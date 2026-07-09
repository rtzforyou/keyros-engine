# Runbook — Apply/Test da migração Team Real Model (PR #9)

Agent: Claude / Fable
Date: 2026-07-08
Migration: `supabase/migrations/20260711000000_team_users_columns_and_signup.sql`
Product PR: rtzforyou/easytattoo-crm#9
Status: NOT APPLIED — este runbook não autoriza o apply; o apply é um gate de Victor/ChatGPT.

> Nota: o inbox referia "Task 1 runbook exists", mas nenhum runbook foi
> encontrado em keyros-engine nem no repo do produto (apenas o esqueleto de
> checklist no corpo do PR #9). Este documento cria o runbook em falta.

## 1. Pré-condições

- [ ] PR #9 revisto e merged em `main` (gate: merge é decisão de Victor/ChatGPT).
- [ ] Confirmar que nenhuma migração remota nova entrou depois de `20260711000000` que toque em `public.users`, `handle_new_user` ou `accept_invitation`.
- [ ] Confirmar as versões actuais em produção antes do apply (para rollback):

```sql
SELECT pg_get_functiondef('public.handle_new_user()'::regprocedure);
SELECT pg_get_functiondef('public.accept_invitation(uuid)'::regprocedure);
SELECT column_name FROM information_schema.columns
WHERE table_schema='public' AND table_name='users';
```

Guardar o output num ficheiro de evidência (agent-room/reports/) antes do apply.

## 2. Apply

Via MCP Supabase (projeto `xqifnqrvcnpxkxftycvf`):

- Tool: `apply_migration`
- Name: `20260711000000_team_users_columns_and_signup`
- Conteúdo: o ficheiro do PR #9, sem alterações.

Não usar `execute_sql` para DDL (tem de ficar registado em `supabase_migrations`).

## 3. Verificação pós-apply (estrutural)

```sql
-- Colunas novas existem, permissions NOT NULL DEFAULT '[]'
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema='public' AND table_name='users'
  AND column_name IN ('permissions','email','avatar_url');

-- Backfill de email: não devem restar linhas com email NULL que tenham par em auth.users
SELECT count(*) FROM public.users u
JOIN auth.users au ON au.id = u.id
WHERE u.email IS NULL;

-- Trigger guard instalado
SELECT tgname FROM pg_trigger
WHERE tgrelid = 'public.users'::regclass
  AND tgname = 'protect_sensitive_user_columns_trg';
```

Esperado: 3 colunas presentes; contagem 0; trigger presente.

## 4. Testes funcionais (na ordem)

Executar com utilizadores de teste dedicados — nunca com contas reais de clientes.

### 4.1 Signup normal sem convite
1. Registar email novo sem convite pendente.
2. Esperado: nova organização criada; linha em `public.users` com `role='admin'`, `permissions='[]'`, `email` preenchido.

### 4.2 Signup com convite pendente
1. Como admin da Org A, criar convite para um email novo (role `staff`, permissions ex. `["dashboard","calendar"]`).
2. Registar com esse email.
3. Esperado: utilizador entra na Org A (nenhuma org nova), `role` e `permissions` copiados do convite, convite marcado `accepted`.

### 4.3 Membro edita o próprio perfil (permitido)
1. Autenticado como membro não-admin, `UPDATE public.users SET full_name='X', avatar_url='...' WHERE id=auth.uid()` (via app ou SQL com JWT do membro).
2. Esperado: sucesso.

### 4.4 Membro tenta auto-escalada (bloqueado)
1. Autenticado como membro não-admin, tentar `UPDATE ... SET permissions='["settings"]'`, depois `SET role='admin'`, depois `SET organization_id=<outra org>`.
2. Esperado: exceção `Not allowed to change role, permissions, or organization_id` nas três tentativas.

### 4.5 Admin edita membro da mesma org (permitido)
1. Autenticado como admin da org, alterar `role`/`permissions` de um membro da mesma org.
2. Esperado: sucesso (RLS "Admins can update organization members" + guard permite admin).

### 4.6 accept_invitation (utilizador já existente)
1. Utilizador autenticado de outra org aceita convite via RPC `accept_invitation(token)`.
2. Esperado: muda de organização, role/permissions do convite aplicados, convite `accepted`. O bypass transaction-local não pode vazar: repetir 4.4 depois deste teste e confirmar que continua bloqueado.

### 4.7 Multi-tenant intacto
1. Membro da Org A faz SELECT de `public.users`.
2. Esperado: só vê membros da Org A.

## 5. Verificação no frontend

Depois do apply + merge do PR #12 (useTeamMembers read-only):
- Aba Equipa mostra membros reais com email/permissions preenchidos.
- Refresh mantém dados reais.

## 6. Rollback

Não-destrutivo (as colunas ficam — são aditivas e inofensivas):

```sql
DROP TRIGGER IF EXISTS protect_sensitive_user_columns_trg ON public.users;
DROP FUNCTION IF EXISTS public.protect_sensitive_user_columns();
-- Restaurar as versões da migration 20260710000000 de:
--   public.handle_new_user()
--   public.accept_invitation(uuid)
-- (copiar o corpo exacto do ficheiro 20260710000000_real_invite_acceptance_flow.sql)
```

Registar o rollback como migração nova (nunca editar migrações passadas).
NUNCA fazer `DROP COLUMN` em rollback — seria destrutivo e precisa de aprovação explícita de Victor.

## 7. Riscos conhecidos

- O guard bloqueia UPDATEs de `role/permissions/organization_id` feitos por não-admins mesmo em fluxos legítimos futuros — qualquer fluxo novo desse tipo precisa do padrão `app.bypass_user_guard` ou de role admin.
- `handle_new_user` faz match de convite por email: emails com capitalização diferente em `auth.users` vs `invitations` podem não fazer match (comportamento herdado da 20260710000000, não introduzido pelo PR #9). Candidato a task isolada futura: normalizar com `lower(email)`.
- Sessões com JWT antigo não são afetadas (guard lê `auth.uid()` em tempo de execução).
