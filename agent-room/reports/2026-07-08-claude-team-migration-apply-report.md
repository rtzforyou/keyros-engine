# Report — Apply da migração Team Real Model (20260711000000)

Task: Apply + verificação da migração do PR #9 (runbook secções 2–4)
Agent: Claude / Fable
Status: completed (apply e verificação estrutural/guard); testes de signup pendentes de verificação manual
Repo: rtzforyou/easytattoo-crm (migração em main @ 541ea2c)
Branch: main (produto) / claude/zone1-team-real-model (este report)
Commit SHA: migração no repo via merge do PR #9 (541ea2c)
PR: easytattoo-crm#9 (MERGED 2026-07-08T14:39Z por Victor)

## Autorização

- Merge do PR #9: feito por Victor directamente no GitHub.
- Apply em produção: autorizado explicitamente por Victor em sessão (pergunta de gate respondida "Sim, aplicar agora"), 2026-07-08.

## O que foi aplicado

Conteúdo exacto de `supabase/migrations/20260711000000_team_users_columns_and_signup.sql` (main @ 541ea2c), via MCP `apply_migration`.

Remote-only change: no — o ficheiro da migração está committed no GitHub (PR #9).
Nota de versão: o registo remoto em `supabase_migrations.schema_migrations` ficou
com version `20260708144838` (timestamp do apply, nome `team_users_columns_and_signup`),
diferente do nome do ficheiro no repo (`20260711000000`). É o comportamento normal
do MCP e consistente com o histórico (ex.: real_invite_acceptance_flow = remoto
20260707131918 vs ficheiro 20260710000000).

## Verificação estrutural (runbook secção 3) — TODAS ✅

- Colunas: `permissions jsonb NOT NULL DEFAULT '[]'`, `email varchar NULL`, `avatar_url text NULL` — presentes.
- Backfill: 0 linhas de `public.users` com email NULL (7/7 preenchidos a partir de auth.users).
- `permissions='[]'` em 7/7 users (todos admins hoje — acesso total por role, lista vazia correcta).
- Trigger `protect_sensitive_user_columns_trg` instalado (único trigger não-interno em public.users).

## Testes funcionais executados (runbook secção 4)

Método: simulação de JWT via `set_config('request.jwt.claims', ..., true)` (transaction-local) em `execute_sql`, com rollback forçado nos caminhos que mutariam dados. Nenhum dado de produção foi alterado.

- **4.4 auto-escalada bloqueada ✅** — JWT de utilizador não-admin (sub sem linha admin) tentou `UPDATE permissions` de outra linha → exceção `Not allowed to change role, permissions, or organization_id` (guard, linha 18).
- **4.5 admin permitido ✅** — JWT do admin 054f83b2 (org 79f6c924) alterou `permissions` do membro 1a36b75b (mesma org): UPDATE passou o guard (rollback forçado com erro controlado; verificação posterior confirmou `permissions=[]` intacto).
- **4.3 self-edit não-sensível ✅** — mesmo JWT fez UPDATE de `full_name` na própria linha sem exceção.

## Não verificado (pendente, requer contas de teste reais)

- 4.1 signup normal sem convite (trigger handle_new_user end-to-end via auth real)
- 4.2 signup com convite pendente
- 4.6 accept_invitation RPC com utilizador autenticado real (incl. não-vazamento do GUC bypass)
- 4.7 multi-tenant SELECT via RLS com JWT real (os testes acima correm como postgres, que não exercita RLS — RLS de public.users está em produção desde 20260620000000, inalterada por esta migração)
- Secção 5 (frontend): aguarda merge do PR #12

Recomendação: Victor faz um signup de teste com email descartável + um convite de teste, ou autoriza a criação de contas de teste dedicadas.

## Riscos pós-apply

- Fluxos futuros que mudem role/permissions/organization_id por não-admins precisam do padrão `app.bypass_user_guard` (documentado na migração).
- Match de convite por email é case-sensitive (herdado; task isolada futura).

## Rollback

Runbook secção 6 (não-destrutivo). Baseline das funções pré-apply capturada em `2026-07-08-claude-team-migration-preapply-evidence.md`.

## Próximo passo

1. Review/merge do PR #12 (useTeamMembers + TeamContent read-only) — gate de Victor/ChatGPT; depois verificar a aba Equipa com dados completos (email agora preenchido).
2. Depois: EPIC Team Permissions Editing (plano no report do EPIC).
