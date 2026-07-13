# ADR-0007 — Keyros Control Center (KCC) — Platform Access Plane

Estado: **Proposto — arquitetura/discovery. Aguarda aprovação antes de implementar.**
Data: 2026-07-10 · Autor: Claude / Fable
Docs completas: produto `docs/kcc/` (KCC0-discovery, KCC1-architecture). Orquestra: KCC EPIC.

## Contexto
Precisamos de um módulo administrativo **da plataforma** (KCC) para operar o Keyros
(tenants, billing, infra, saúde). Acesso só a papéis de plataforma
(`platform_owner`, `platform_admin`, `developer`, `support`) — nunca utilizadores comuns.
Todo o produto é multi-tenant com RLS `organization_id = get_user_org_id()`. O KCC
precisa de acesso **cross-tenant**, o oposto da RLS.

## Decisão
**Criar um plano de acesso de plataforma SEPARADO e ortogonal**, sem tocar na RLS de
tenant:
1. **Identidade de plataforma desacoplada** — `platform_users` (papel de plataforma
   independente de `users.role`/organização). Bootstrap seguro (seed do 1º owner;
   nunca criável pelo fluxo de convites de org).
2. **Autorização só no backend** — `is_platform_member(min_role)`. O frontend nunca
   liberta acesso; o menu deriva da autz da API.
3. **Cross-tenant só via edge functions de plataforma** (`verify_jwt=true`,
   convenção `platform-*`) que: autenticam → verificam papel → **auditam** →
   consultam com `service_role` (bypassa RLS por design, atrás da autz) → devolvem
   read models agregados/redigidos. Sem `service_role` no cliente; sem PostgREST
   direto a tabelas de plataforma.
4. **RLS de produto INALTERADA** — nada de cláusulas `OR is_platform()` nas tabelas de
   tenant. Se o plano de plataforma falhar, o isolamento de tenant mantém-se.
5. **Auditoria própria** — `platform_audit_logs` regista todo o acesso antes de servir.
6. **Segredos só como estado** — `Configured/Missing/Expired/Rotation Required`; nunca
   valores/prefixos/hash.

## Alternativa rejeitada
Afrouxar a RLS do produto com `USING (org = get_user_org_id() OR is_platform())`:
acopla os planos e um bug no check de plataforma vaza dados cross-tenant. Rejeitado.

## Fases
- **V1 Read-only** (sem ações destrutivas). **V2 Operações** (suspender tenant,
  impersonation segura auditada/limitada, subscription repair, usage reset, support
  mode). **V3 Live monitoring** (realtime/logs/metrics/alerts).

## Consequências
- (+) Isolamento multi-tenant preservado; superfície de plataforma auditável e mínima.
- (+) Papéis de plataforma reutilizáveis em CI/observabilidade futura.
- (−) Novo plano a manter (tabelas + edge functions + auditoria); métricas de
  Health/Usage exigem instrumentação nova (hoje não há APM — ver CAP0).

## Gate
Discovery/arquitetura apenas. **Nenhum schema, edge function, frontend, deploy ou
merge** até aprovação. Não altera Product Core, PR #54 nem Project Memory.
