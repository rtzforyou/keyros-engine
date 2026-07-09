# Report — Slice 1 apply + execução da suite RLS

Task: Aplicar migrações da Slice 1 e correr os testes RLS (comando de Victor "start test rls")
Agent: Claude / Fable
Status: completed — 10/10 testes PASS; dados de teste removidos; produção intacta
Repo: rtzforyou/easytattoo-crm
Branch: claude/team-permissions-editing
Commit SHA: 6e0911e (migração reforçada) — ver PR #13
Projeto Supabase: xqifnqrvcnpxkxftycvf

## Autorização

Victor autorizou explicitamente em sessão ("Apply + testar tudo"): aplicar as 3
migrações da Slice 1 E criar dados de teste em produção. Override consciente do
"Not approved yet: Apply migrations" do NEXT.md — registado aqui para ChatGPT.

## Achado durante a preparação (reforço da migração)

Ao capturar o baseline das policies de `invitations` apareceu, além da
`Tenant isolation` (FOR ALL), a policy legacy **`Anyone can read pending
invitations`** (SELECT para `public`, da 20260618000003) — expunha email+token
de todos os convites pendentes de todas as orgs. Como policies de SELECT se
combinam por OR, a reescrita admin-only ficaria neutralizada na leitura.
Reforcei `20260712000002` para dropar também essa policy (commit 6e0911e).
Nenhum fluxo depende dela (UI autenticada usa SELECT org-scoped; ecrã anónimo
usa RPC get_invitation_by_token). Confirmado por grep no código.

## Migrações aplicadas (ordem canónica)

Registos remotos em supabase_migrations:
1. `20260708171335` normalize_legacy_module_permissions
2. `20260708171355` team_permissions_editing_support (last-admin + audit INSERT policy)
3. `20260708171409` invitations_admin_only_write (fix + remoção da leitura pública)

Ficheiros no repo: 20260712000000/1/2. (Divergência de timestamp nome-ficheiro
vs registo remoto é o comportamento normal do MCP, consistente com o histórico.)

## Dados de teste (criados e REMOVIDOS)

Org A (`__RLS_TEST_ORG_A__`) com admin + membro staff; Org B com admin; 1
convite pendente na org A. Emails `rlstest-*@keyros-test.local`. Setup usou o
trigger de signup + DELETE/INSERT para posicionar o membro na org A sem
disparar o last-admin. Tudo removido no fim — verificado: 0 users/orgs/invites
de teste; `public.users` de volta a 7 (estado original); nenhuma permissão de
produção alterada (`permissions <> '[]'` = 0).

## Resultado da suite (10/10 PASS)

| # | Teste | Esperado | Resultado |
|---|---|---|---|
| T1 | membro cria convite admin | BLOQUEADO | BLOQUEADO ✅ (fix crítico) |
| T2 | membro revoga convite | 0 linhas | 0 linhas ✅ |
| T3 | admin cria convite válido | PERMITIDO | PERMITIDO ✅ |
| T4 | admin de outra org cria convite na org A | BLOQUEADO | BLOQUEADO ✅ |
| T5 | admin de outra org vê convites da org A | 0 visíveis | 0 ✅ (leitura pública fechada) |
| T5b | membro vê convites da própria org | ≥1 | 2 ✅ |
| T6 | membro auto-escala as próprias permissions | BLOQUEADO | BLOQUEADO ✅ (guard) |
| T7 | despromover o último admin da org | BLOQUEADO | BLOQUEADO ✅ (last-admin) |
| T8a | admin insere audit log próprio | PERMITIDO | PERMITIDO ✅ |
| T8b | membro insere audit log | BLOQUEADO | BLOQUEADO ✅ |

Método: bloco plpgsql único, cada teste numa subtransação com JWT simulado
(`set_config('request.jwt.claims')`) + `SET LOCAL ROLE authenticated`, captura
de PASS/FALHA numa tabela temporária. O fix crítico (escalada indireta de
privilégios via convite) está fechado e provado.

## Estado pós-apply verificado

- invitations: 4 policies (SELECT membros; INSERT/UPDATE/DELETE admin). Legacy removidas.
- audit_logs INSERT: policy admin + service_role (ambas presentes).
- Produção: 7 users intactos, permissions `[]`, backfill de email preservado.

## Riscos / notas

- As migrações estão AGORA aplicadas em produção. O PR #13 deixa de ser
  "migrations NOT applied" — atualizado o comentário do PR.
- Mudança de comportamento agora LIVE no backend: membros não-admin já não
  conseguem criar/alterar/revogar convites (bloqueio de RLS). A UI do PR #13
  (que esconde essas acções) ainda NÃO está em main — até o PR #13 mergear, um
  membro não-admin vê os botões mas recebe erro ao clicar. Recomendo priorizar
  o merge do PR #13 ou comunicar. (Produção só tem admins hoje, logo impacto
  real nulo até existir um membro não-admin.)
- accept_invitation/handle_new_user (SECURITY DEFINER) não afectados — o fluxo
  de aceitação de convite continua a funcionar.

## Rollback

Se necessário: migração nova que recria as policies antigas de invitations e
restaura o guard/normalização (headers das migrações + tabela no comentário do
PR #13). Não recomendado — reabre a vulnerabilidade.

## Próximo passo

Merge do PR #13 (UI) — gate de Victor/ChatGPT. Depois: Slice 2 (Billing
foundation).
