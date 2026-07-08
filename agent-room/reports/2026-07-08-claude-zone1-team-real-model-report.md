# Report — EPIC ZONE 1 Team Real Model

Task: EPIC ZONE 1 — Team Real Model (Sprints 1–7)
Agent: Claude / Fable
Status: needs_review
Repo: rtzforyou/easytattoo-crm + rtzforyou/keyros-engine
Branch: `claude/team-members-hook-readonly` (produto) / `claude/zone1-team-real-model` (engine)
Commit SHA: produto `a2b493b3afdbfcbd6e140a44328356a6ba054d37`; engine: ver PR desta branch
PR: easytattoo-crm#12 (draft) — https://github.com/rtzforyou/easytattoo-crm/pull/12

## Sprint 1 — PR #9 readiness: VERIFICADO ✅

- PR #9 OPEN, `mergeable: MERGEABLE`, `mergeStateStatus: CLEAN` contra main@`82cc656`.
- Único ficheiro: `supabase/migrations/20260711000000_team_users_columns_and_signup.sql`; timestamp ordena depois da última migração de main (`20260710000000`). Sem colisão.
- Dependências verificadas contra main: `invitations.permissions` existe (20260618000003); as funções `handle_new_user`/`accept_invitation` que o PR redefine são as versões de 20260710000000 — substituição consistente (acrescenta email/permissions, mantém a semântica invite-aware).
- RLS relevantes em `public.users` (20260620000000) confirmadas: SELECT por org, UPDATE own profile, UPDATE por admin — exactamente o cenário que o guard do PR #9 endereça.
- Nota de paralelismo: existem migrações locais não commitadas no working tree de Victor (20260705–20260708, campanhas/circuit breaker/rate limit) com timestamps ANTERIORES a 20260710/20260711. Se forem commitadas/aplicadas depois, o Supabase aceita, mas ChatGPT deve verificar a ordem de apply quando esse workstream chegar a PR.

Veredicto: PR #9 continua merge-ready. Merge é gate de Victor/ChatGPT.

## Sprint 2 — Runbook apply/test: CRIADO (não existia) ✅

O inbox afirmava "Task 1 runbook exists", mas não há runbook em keyros-engine nem no produto — só o esqueleto de checklist no corpo do PR #9. Também não encontrei o documento "V0 production verification" referido no inbox.

Entregue: `agent-room/reports/2026-07-08-claude-team-migration-apply-runbook.md` — pré-condições, apply via MCP `apply_migration`, verificação estrutural SQL, 7 testes funcionais (signup normal, signup com convite, self-update permitido, auto-escalada bloqueada, admin update, accept_invitation com verificação de não-vazamento do bypass, multi-tenant), rollback não-destrutivo, riscos.

Migração NÃO aplicada (gate respeitado).

## Sprint 3 — useTeamMembers hook: ENTREGUE (draft) ✅

`hooks/useTeamMembers.ts` no PR #12 (draft):
- Resolve `organization_id` do perfil autenticado; bloqueia a query se ausente.
- `.eq('organization_id', organizationId)` explícito + RLS como segunda camada.
- Retorna `{ teamMembers, isLoading, error, refetch }`.
- Mapeamento defensivo de `email/permissions/avatar_url` via `select('*')`: funciona ANTES do apply do PR #9 (campos caem em fallback vazio) e fica completo DEPOIS, sem alterar código. Por isso o PR não parte nada se for merged antes do apply — mas fica draft e marcado "Blocked by #9 apply" para dados completos.

## Sprint 4 — TeamContent read-only: ENTREGUE (draft, mesmo PR) ✅

- Lista de membros passa a dados reais do hook; remove a dependência `useMockData` de `components/team/TeamContent.tsx` (menos um consumidor do hook legado).
- Estados loading/error/empty adicionados; avatar com fallback de inicial.
- Read-only estrito: dropdown editar/remover membro e dialog de edição REMOVIDOS da UI até à slice de edição de permissões. Convites e artistas internos intactos.

## Sprint 5 — Plano: edição de role/permissions por admin

1. **Registry estável de módulos** (pré-requisito): hoje `MODULES` em TeamContent tem `aiAgent` mas não `team`; `domains/team.md` lista `team` e não `aiAgent`. Criar `lib/moduleRegistry.ts` com IDs estáveis em inglês (fonte única para convites, edição e enforcement) e alinhar com ChatGPT qual é a lista canónica.
2. `useTeamMembers.updateTeamMember(id, { role, permissions })`: UPDATE em `public.users`, permitido só a admins — defesa em 3 camadas: UI (esconder acção para não-admin), RLS "Admins can update organization members", trigger guard do PR #9.
3. Restaurar o dialog de edição (o JSX removido no Sprint 4 está no histórico git) ligado ao hook, com validação de IDs contra o registry.
4. **Protecção last-admin**: impedir despromover/remover o último admin da org. Idealmente no DB (mesma função guard ou constraint via trigger); no mínimo na UI + validação no hook. Decisão de design para ChatGPT: DB-side (recomendado) vs frontend-only.
5. **Auditoria** (exigida por domains/team.md): registar mudanças de role/permissions em `activity_logs` (org_id, actor, alvo, old→new). Sem tabela nova; usar a existente.
6. Dependências: PR #9 aplicado (coluna `permissions`); PR #12 merged.

## Sprint 6 — Plano: enforcement de permissions no Login

1. No bootstrap de auth (App.tsx), após resolver a sessão, ler a própria linha de `public.users`: `role`, `permissions` (+ `status` quando o Sprint 7 aterrar).
2. Resolução de acesso:
   - `role='admin'` → todos os módulos activos do registry (nunca depende da lista).
   - não-admin → intersecção `permissions` × registry de módulos activos.
3. Aplicar em: sidebar (esconder módulos sem acesso) e no router manual do App.tsx (bloquear navegação directa — não basta esconder o botão).
4. Fallback seguro: erro ao ler permissions → tratar como sem acesso a módulos não-essenciais, com erro visível (nunca conceder tudo em falha).
5. Enforcement frontend é UX, não segurança: a segurança real continua a ser RLS por tabela/org. Módulos com dados sensíveis próprios (ex. payments) podem precisar de RLS adicional por permissão — fora deste escopo, avaliar caso a caso.
6. Pré-condições explícitas do inbox respeitadas: só implementar depois de migração aplicada + hook + read-only UI verificados. NÃO implementado agora.

## Sprint 7 — Design: removeMember soft-remove

1. Sem DELETE. Migração futura adiciona a `public.users`: `status varchar(20) NOT NULL DEFAULT 'active'` (valores `active`/`revoked`) + `removed_at timestamptz`.
2. `status` entra na lista de colunas sensíveis do trigger guard (só admin muda; mesmo padrão do PR #9).
3. Remoção = admin faz `status='revoked', removed_at=now()`. Linha preservada (auditoria, FKs de deals/appointments/tattooer_id continuam válidas).
4. Login/enforcement (Sprint 6) trata `status<>'active'` como sem acesso: sessão bloqueada com mensagem, sign-out forçado.
5. RLS SELECT continua a mostrar revogados na lista de equipa (badge "Removido") — decisão de UI para Victor: mostrar ou filtrar por defeito.
6. Re-entrada: novo convite para o mesmo email reactiva (`status='active'`, role/permissions do convite) via fluxo accept_invitation — sem linha duplicada (id = auth.uid é PK).
7. Hard delete: não autorizado; se um dia for pedido, exige decisão explícita de Victor + plano para FKs órfãs e `auth.users`.

## Verificação realizada

- Build (worktree em origin/main + PR #12): `npm run build` ✓.
- Typecheck: `npx tsc --noEmit` — 0 erros nos ficheiros alterados; 3 erros pré-existentes em main, não relacionados (ver Riscos).
- Manual/browser: NÃO verificado — exige sessão Supabase autenticada; runtime fica para verificação pós-apply (secção 5 do runbook).
- Database/RLS: nenhuma mudança remota feita. Migração NÃO aplicada.

Remote-only change: no.

## Keyros Engine updated

- yes — este report + runbook (`agent-room/reports/`), nesta branch.
- `domains/team.md` não alterado (ownership do ChatGPT; os planos acima propõem, não decidem).

## Riscos / não verificado

- **Main tem typecheck quebrado (pré-existente, Zone 2/Antenor)**: `components/contacts/ContactDetailsDrawer.tsx` importa `./ContactFormsTab` que já não existe (resíduo da remoção de Forms; o ficheiro está órfão, por isso o build vite passa). Também `hooks/useLeads.ts` e `hooks/useMockData.ts` com `stageId` em falta no mapeamento de `Deal`. Encaminhar para Antenor.
- Runtime do PR #12 não testado em browser (sem sessão autenticada disponível); o comportamento pré-migração assenta em `select('*')` + fallbacks — verificar visualmente após deploy de preview ou apply.
- Case-sensitivity no match de convites por email (herdado da 20260710000000) — candidato a task isolada.
- Documento "V0 production verification" referido no inbox não foi encontrado — confirmar com ChatGPT onde vive.

## Rollback

- PR #12: fechar PR / reverter merge — sem efeitos em DB.
- Migração PR #9: secção 6 do runbook (rollback não-destrutivo, funções restauradas via migração nova).

## Next isolated task recommended

1. Victor/ChatGPT: merge PR #9 → apply da migração seguindo o runbook (gate).
2. Depois: review/merge PR #12 e verificação visual da aba Equipa.
3. Em paralelo (Antenor): reparar typecheck de main (ContactDetailsDrawer órfão + stageId em useLeads/useMockData).
4. Próximo EPIC recomendado: ZONE 1 — Team Permissions Editing (Sprint 5 do plano acima, inclui module registry + auditoria).

Permission requested from Victor: autorização para (a) merge do PR #9 e apply da migração via runbook; (b) decisão UI do Sprint 7 (mostrar vs esconder membros revogados); (c) lista canónica de módulos do registry.
