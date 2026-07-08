# Report — Slice 1 implementation (NEXT.md 1e56df6, tarefas 1–20)

Task: Slice 1 — Team Permissions Editing + invitations security fix (IMPLEMENTATION FIRST)
Agent: Claude / Fable
Status: needs_review — código completo em PR draft; parado no gate merge/apply (tarefa 20)
Repo: rtzforyou/easytattoo-crm
Branch: claude/team-permissions-editing
Commit SHA: 5991846c939b0c1ea266a7cb7d114acfa21700c8 (+ f978210 anterior)
PR: #13 (draft) — comentário com ordem de apply e rollback publicado

## Estado por tarefa

| # | Tarefa | Estado |
|---|---|---|
| 1 | Fix no PR 13 | ✅ mesmo PR, commit 5991846 |
| 2–4 | INSERT/UPDATE/DELETE admin-only | ✅ migração `20260712000002_invitations_admin_only_write.sql` (NÃO aplicada) |
| 5–7 | Testes membro-não-convida / admin-convida / cross-org | ✅ suite `supabase/tests/team_invitations_rls_test.sql` (T1–T5; execução BLOCKED: pós-apply + exige membro não-admin de teste — produção só tem admins, criar conta é production data change = gate) |
| 8 | Backend slice em PR draft | ✅ completo (registry, hook, dialog, migrações) |
| 9 | Last-admin protection | ✅ já em `20260712000001` (T7 na suite) |
| 10 | Audit logging pós-migração | ✅ código pronto; INSERT policy na `20260712000001` (T8a–c na suite); até ao apply o audit no-opa com log de consola |
| 11 | Registry como constantes TS | ✅ mantido; gatilhos para tabela documentados (design pack 2 §17) |
| 12 | Normalização idempotente | ✅ `20260712000000` (UPDATE mapeia+deduplica; re-executável) |
| 13 | plan_id NULL = comportamento actual | ✅ implementado em `lib/entitlements.ts` (resolveOrgEntitlements(null) → todos os active + seats ilimitados); confirmação em produção só após migração de plans existir |
| 14 | get_org_entitlements draft | ✅ TS `resolveOrgEntitlements` (sem provider logic; versão SQL fica para a migração de plans — hoje não há tabela para referenciar) |
| 15 | Resolver de acesso efectivo | ✅ TS `resolveEffectiveModules` + `hasModuleAccess` (admin→plan; membro→permissions∩plan, não-atribuíveis excluídos) |
| 16 | Ordem de apply | ✅ 000000 → 000001 → 000002 (sem dependência funcional; ordem canónica por timestamp) — no comentário do PR |
| 17 | Rollback por migração | ✅ headers das migrações + tabela no comentário do PR (000000 one-way inócua; 000001/000002 revertíveis por migração nova) |
| 18 | Build/typecheck | ✅ tsc sem erros novos (3 pré-existentes de main); vite build ✓ |
| 19 | Report | ✅ este ficheiro |
| 20 | Stop no gate | ✅ nada merged, nada aplicado |

## Extra (fora da lista, dentro do âmbito)

- TeamContent: botão de convite + dropdown de acções de convite escondidos
  para não-admins (rosto UI da nova policy; o servidor bloqueia sempre).

## Riscos / não verificado

- Testes RLS (T1–T8) não executados: dependem do apply (gate) e de um membro
  não-admin de teste (gate de dados). Pedido a Victor: autorizar criação de
  2 contas de teste (1 admin org B, 1 membro org A) OU correr a suite ele mesmo.
- Runtime browser não verificado (sem credenciais); preview Cloudflare do PR compila.
- Após apply da 000002, um membro não-admin deixa de conseguir convidar — é
  mudança de comportamento INTENCIONAL (fecho de vulnerabilidade); comunicar.

## Pedido de gate (tarefa 20)

Autorização para: (a) merge PR #13; (b) apply 000000→000001→000002; (c) criação
de contas de teste para executar a suite T1–T8 e a verificação visual.
