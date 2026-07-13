# ADR-0008 — Partner Platform — Consented Cross-Tenant Plane

Estado: **Proposto — arquitetura/discovery. Aguarda aprovação antes de implementar.**
Data: 2026-07-10 · Autor: Claude / Fable
Docs completas: produto `docs/partner-platform/` (PP0-discovery, PP1-architecture).

## Contexto
Um parceiro (agência/rede/revendedor) precisa de gerir **múltiplos estúdios** com uma só
sessão (consolidado + drill sem logout), mantendo o isolamento multi-tenant. O produto é
100% RLS por org (`organization_id = get_user_org_id()`); 1 login = 1 org. Partner **não é**
tenant, org, nem substitui o KCC.

## Decisão
**Criar um TERCEIRO plano — partner plane — cross-tenant SCOPED e CONSENTIDO**, sem tocar
na RLS de tenant:
1. **Entidade desacoplada:** `partners` + `partner_users` (papel de parceiro independente
   de `users.role`/org).
2. **Ligação consentida:** `partner_workspace_links` (partner↔org, estado, permissões por
   módulo, modo de billing). **O estúdio autoriza** (convite + aceitação do owner); o
   parceiro nunca reivindica orgs. Revogar o link corta o acesso.
3. **Autorização backend + escopo:** `is_partner_member(partner_id, min_role)` **E**
   org-alvo ∈ links ativos **E** permissão de módulo. Frontend nunca autoriza.
4. **Cross-tenant só via camada partner:** edge functions `partner-*` (`verify_jwt=true`,
   `service_role` atrás da autz, **filtradas ao conjunto ligado**), **auditadas**
   (`partner_audit_logs`). Nunca service_role no cliente; nunca PostgREST cross-org direto.
5. **RLS de produto INALTERADA** e **nunca reutilizar `organization_admin`**.
6. **Permissões por módulo:** Nenhum/Leitura/Escrita/Administração, limitadas pelo que o
   estúdio consente e reduzidas pelo `partner_role`.

## Distinção vs KCC (ADR-0007)
KCC = plataforma, vê **todos** os tenants, sem consentimento. Partner = cliente, vê um
**subconjunto consentido**, com aceitação do estúdio. Planos e auditorias separados.

## Alternativa rejeitada
Afrouxar a RLS do produto (`OR is_partner()`) ou reutilizar org admin — acopla planos e
arrisca vazamento/escalada cross-tenant. Rejeitado.

## Roles
`partner_owner` > `partner_admin` > `partner_manager` > `partner_viewer`.

## Roadmap
PP0 Discovery · PP1 Architecture · PP2 Database · PP3 Permissions · PP4 Dashboard ·
PP5 Billing (studio-paga / partner-paga / comissão / licenciamento) · PP6 Analytics ·
PP7 Support Tools.

## Consequências
- (+) Isolamento preservado; parceiros habilitados sem quebrar o produto; base para
  agências/franquias/revenda.
- (−) Novo plano a manter (tabelas + edge functions + auditoria + consentimento);
  billing multi-modo é decisão comercial pendente.

## Riscos
Consentimento/revogação, fuga de escopo (filtrar sempre pelos links), escalada de papel,
ambiguidade de billing por estúdio. Mitigações no PP1.

## Gate
Discovery/arquitetura apenas. **Nenhum schema, edge function, frontend, deploy ou merge**
até aprovação. Não altera Product Core, Project Memory, PR#54 nem KCC.
