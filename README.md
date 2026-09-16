[README.md](https://github.com/user-attachments/files/32312547/README.md)
# Supabase — Apontamento de Horas

## Projeto
- Nome: apontamento-horas
- Organização: AG Digital
- Região: sa-east-1 (São Paulo)
- Plano: gratuito (US$ 0/mês)
- URL da API: https://zsrtrkqnpjdjsyyvflrf.supabase.co
- Chave pública (publishable): sb_publishable_N4tQ5Q8uTTJYfLj2q5Vm_A_QbvtF3rZ
  - Pode ir no HTML. A segurança vem das regras de acesso (RLS).
  - Nunca colocar a chave service_role no HTML.

## Fase 1 — concluída (migração `schema_inicial_apontamento`)
| Tabela | Conteúdo |
|---|---|
| perfis | Nome, e-mail, jornada, expediente, intervalo, início do chat, ferramenta (Liveops), feriados desativados, facultativos ativos, lembretes (liga/desliga e horário), fuso |
| chamados | Chamados cadastrados; um único padrão por usuário |
| apontamentos | Data, início e fim em minutos (0–1440), chamado, descrição, manual, he, origem |
| feriados_personalizados | Feriados extras do usuário |
| lancamentos_ferramenta | Marcação "lançado no Liveops", com totais do dia no momento da marcação |
| cronometro | Cronômetro em andamento, sincronizado entre dispositivos |
| notificacoes_enviadas | Log dos lembretes; gravado só pela função agendada |

Regras no banco:
- RLS em todas as tabelas: cada usuário só vê e altera os próprios dados. O acesso anônimo foi revogado.
- Sobreposição de horário no mesmo dia é bloqueada por uma exclusion constraint.
- O fim do apontamento precisa ser maior que o início.
- Ao criar um usuário, o banco cria o perfil e o chamado padrão 476077.
- `definir_chamado_padrao(id)` troca o chamado padrão em uma única operação.

Testes executados (com rollback): sobreposição bloqueada; inserção em nome de outro usuário bloqueada; fim < início bloqueado; usuário não grava log de notificação; troca de chamado padrão; isolamento entre dois usuários. Advisor de segurança sem apontamentos.

## Fase 2 — programa com login (pronto para publicar)
Arquivo: `web/index.html`, versão online do programa. A versão offline `apontamento-horas.html` continua na pasta como alternativa.

O que mudou em relação à versão offline:
- **Acesso:** tela de login com e-mail e senha, recuperação de senha por e-mail e botão Sair. Novos cadastros ficam fechados (uso individual).
- **Dados:** gravados no Supabase. Cada alteração vai para a nuvem logo em seguida. Um indicador no topo mostra "Salvando…", "Salvo na nuvem" ou "Erro ao salvar".
- **Falha ao salvar:** aparece um aviso com "Tentar de novo" ou "Descartar e recarregar do servidor".
- **Outros dispositivos:** ao voltar para a aba, o programa recarrega os dados, porque outro dispositivo pode ter alterado algo. O cronômetro em andamento também sincroniza.
- **Nova área:** "Meu perfil e lembretes" em Configurações, com nome e lembretes de início e fim do dia (liga/desliga e horário).
- **Importação:** o backup JSON da versão offline pode ser importado. IDs antigos são convertidos.
- **Regras sem mudança:** HE, feriados, chat, relatórios e marcação do Liveops funcionam como antes.

Testado com um Supabase simulado: login, lançamento pelo chat, HE, marcação do Liveops, virada da meia-noite, chamado padrão, perfil, feriado personalizado, cronômetro, erro e nova tentativa, recarga da página, importação de backup e saída.

### Passos no painel do Supabase (uma vez)
1. **Criar o seu usuário:** Authentication → Users → Add user → Create new user. Informe e-mail e senha e marque **Auto Confirm User**. O banco cria sozinho o perfil e o chamado padrão 476077.
2. **Fechar novos cadastros:** Authentication → Sign In / Providers → desligue **Allow new users to sign up**.
3. **Liberar o link de recuperação de senha** (depois de publicar no GitHub Pages): Authentication → URL Configuration.
   - **Site URL:** `https://<seu-usuario>.github.io/apontamento-horas/`
   - **Redirect URLs:** adicione a mesma URL.

### Publicar no GitHub Pages
1. **Criar o repositório:** no GitHub, crie um repositório público chamado `apontamento-horas`.
2. **Enviar o arquivo:** Add file → Upload files → envie o `web/index.html` com o nome `index.html` → Commit.
3. **Ativar o Pages:** Settings → Pages → Source: *Deploy from a branch* → Branch `main`, pasta `/ (root)` → Save.
4. **Acessar:** em 1 a 2 minutos, a página fica em `https://<seu-usuario>.github.io/apontamento-horas/`.

A chave que está no HTML é a chave pública (publishable), feita para ficar no navegador. Os dados são protegidos pelo login e pelas regras de acesso (RLS) de cada usuário.

### Levar os dados da versão offline
1. Na versão offline: Configurações → Exportar backup (JSON).
2. Na versão online: Configurações → Importar backup → Juntar ou Substituir tudo.

## Próximas fases
3. Lembretes por e-mail: Edge Function + pg_cron + Resend. Precisa da conta no Resend e da chave de API.
4. Testes de ponta a ponta com o site publicado.
