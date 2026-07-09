# Security review — plano (Team, módulos, planos, access checks)

Task: NEXT.md Claude tarefa 10
Agent: Claude / Fable
Status: needs_review (plano de revisão; execução é EPIC próprio)

## Âmbito

Superfícies tocadas pelo Team Real Model + Module Registry + Billing futuro.

## 1. Team / public.users

- [ ] RLS: SELECT restrito à org (testado 2026-07-08 com JWT real ✅ — repetir após cada migração de users).
- [ ] Guard trigger: matriz completa de UPDATE — {admin, não-admin, sem-linha, service_role, bypass GUC} × {role, permissions, organization_id, full_name, status futuro}. Testes 4.4/4.5/4.3 já passaram; falta matriz completa e não-admin REAL (produção só tem admins).
- [ ] Last-admin protection (migração 20260712000001, quando aplicada): tentar despromover único admin (deve falhar), com 2 admins (deve passar), via accept_invitation com bypass (deve falhar na saída do último admin).
- [ ] accept_invitation: GUC bypass não vaza entre transações; convite expirado/revogado rejeitado; email de outro utilizador não aceitável.
- [ ] handle_new_user: signup com convite pendente não cria org nova; sem convite cria org isolada. Case-sensitivity de email (risco conhecido) — decidir normalização lower().
- [ ] Invitations: token UUID único, expira, single-use (status accepted), RLS org-scoped, get_invitation_by_token não expõe mais do que o necessário.

## 2. Module Registry / permissions

- [ ] `users.permissions` só muda por admin (guard) — confirmar que não existe nenhum outro caminho de escrita (edge functions, RPCs).
- [ ] normalizeModuleIds: ids desconhecidos são descartados (nunca concedem acesso); aliases só em leitura.
- [ ] Enforcement é DB-first: frontend esconder ≠ proteger — cada domínio continua protegido por RLS das suas tabelas mesmo se a UI falhar.
- [ ] audit_logs: INSERT policy (20260712000001) só admin/própria org/em nome próprio; SELECT já admin-only; verificar que details não guarda PII desnecessária.

## 3. Plans / Billing (quando implementado)

- [ ] plans: escrita só service_role; entitlements não editáveis por clientes.
- [ ] organizations.plan_id: só service_role/backoffice muda (nunca o cliente via RLS UPDATE de organizations — REVER a policy "Admins can update their own organization": hoje um admin pode fazer UPDATE da própria org; quando plan_id existir, essa policy tem de excluir plan_id ou ganhar guard por coluna, senão qualquer admin se auto-upgrada). **Achado preventivo mais importante deste plano.**
- [ ] Seat limit trigger: não contornável por INSERT direto (RLS de invitations), race de convites simultâneos (usar contagem na transação; aceitável para o volume actual).
- [ ] Falha de leitura de entitlements no frontend → negar não-core, nunca conceder.

## 4. Endpoints públicos e service_role (regressão)

- [ ] Re-correr checklist CLAUDE.md: landing-lead (token, rate-limit, allowlist, org_id do payload ignorado), webhooks HTTP 200, service_role ausente do bundle (`grep` no dist), envs VITE_* limpos.
- [ ] Edge functions com service_role validam organization_id manualmente (automation-*, whatsapp-*).

## 5. Execução proposta

1. Sessão de testes com 2 orgs de teste + 1 membro não-admin real (pedir a Victor a criação, ou script com contas descartáveis).
2. Matriz da secção 1 via SQL simulado (JWT set_config) + browser real.
3. Relatório com achados classificados (crítico/alto/médio) e correcções como tasks isoladas.

Recomendação imediata (antes até da review): tratar o item 3.2 (policy de
UPDATE de organizations vs futuro plan_id) na migração que criar plan_id.
