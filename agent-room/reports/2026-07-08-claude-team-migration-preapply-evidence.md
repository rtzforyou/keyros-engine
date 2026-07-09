# Evidência pré-apply — migração Team Real Model (20260711000000)

Agent: Claude / Fable
Date: 2026-07-08 (após merge do PR #9 em main @ 541ea2c)
Projeto Supabase: xqifnqrvcnpxkxftycvf
Runbook: 2026-07-08-claude-team-migration-apply-runbook.md (secção 1)

## Estado de produção verificado (read-only)

### public.users — colunas atuais

```
id               uuid        NOT NULL
organization_id  uuid        NOT NULL
role             varchar     NOT NULL DEFAULT 'admin'
full_name        varchar     NULL
created_at       timestamptz NOT NULL DEFAULT timezone('utc', now())
```

→ `permissions`/`email`/`avatar_url` NÃO existem ainda. Migração não aplicada. ✅

### supabase_migrations — última aplicada

```
20260707131918  real_invite_acceptance_flow
```

→ Nenhuma migração posterior; nada toca users/handle_new_user/accept_invitation depois desta. ✅

Nota: as migrações locais do outro workstream (bulk campaigns 20260705095211,
circuit breaker 20260705103118, rate limiting 20260706134727/20260706141839,
secure_appointments 20260706154621) JÁ constam como aplicadas em produção —
são remote-first e os ficheiros correspondentes existem no working tree local
de Victor ainda não commitados. Sem conflito com a migração Team.

### Funções em produção (baseline para rollback)

`handle_new_user()` e `accept_invitation(uuid)` em produção correspondem
exatamente às versões da migração `20260710000000_real_invite_acceptance_flow.sql`
(INSERT sem email/permissions; UPDATE sem permissions; sem GUC bypass).
O rollback é portanto: restaurar os corpos desse ficheiro + DROP do trigger/função
guard, via migração nova (runbook secção 6).

### Trigger guard

`protect_sensitive_user_columns_trg` ainda não existe em produção (esperado).

## Conclusão

Todas as pré-condições da secção 1 do runbook verificadas. Pronto para apply
via `apply_migration` com o conteúdo exato de
`supabase/migrations/20260711000000_team_users_columns_and_signup.sql` (main @ 541ea2c).

Apply aguarda autorização explícita de Victor (gate).
