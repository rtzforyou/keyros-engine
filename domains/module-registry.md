# Module Registry — fonte de verdade canónica

Status: implementado em código (PR easytattoo-crm#13, draft)
Implementação: `lib/moduleRegistry.ts` no repo do produto
Especificação de origem: agent-room/reports/2026-07-08-claude-module-registry-v2.md
Aprovado por: ChatGPT/Victor (NEXT.md + MASTER_STATE.md do keyros-orquestra-, 2026-07-08)

## Regra

Nenhum agente inventa ids de módulo. Ids novos entram primeiro aqui e em
`lib/moduleRegistry.ts`, com decisão registada.

## Domínios (ids estáveis)

| id | status | teamAssignable | planControlled | dependsOn | notas |
|---|---|---|---|---|---|
| dashboard | active | sim | não (core) | — | |
| crm | active | sim | sim | — | engloba contacts, deals, pipeline, activities |
| calendar | active | sim | sim | — | |
| messages | active | sim | sim | — | |
| automations | active | sim | sim | messages | |
| team | active | sim ⚠ | sim (seats) | — | |
| billing | active | **não** | não (core) | — | admin-only: plano/seats/subscrição/faturas |
| finance | active | sim | sim | — | engloba payments e expenses do estúdio |
| reports | reserved | não | sim | dashboard | sem superfície própria ainda |
| settings | active | sim ⚠ | não (core) | — | |
| agents | active | sim | sim | messages | ex-"AI" |

⚠ = atribuição exige aviso na UI (poder de configuração).
`dependsOn` é soft (UI avisa; enforcement não bloqueia).

## Aliases legados (leitura apenas; nunca escritos)

```
contacts|deals|pipeline -> crm
payments|expenses       -> finance
aiAgent|ai              -> agents
```

Normalização de dados persistidos: migração draft
`20260712000000_normalize_legacy_module_permissions.sql` (PR #13, não aplicada).

## Consumidores

- Permissões de equipa: convite + edição (TeamContent via TEAM_ASSIGNABLE_MODULES) — implementado no PR #13.
- Login enforcement: admin → módulos do plano; não-admin → permissions ∩ plano (planeado, não implementado).
- Plan entitlements/billing: `plans.entitlements.modules` (planeado — ver 2026-07-08-claude-billing-entitlements-architecture.md).
- Feature flags e acesso de agentes/automações: futuro, sempre por id deste registry.

## Separação id vs label

Labels de display NUNCA participam em lógica. `defaultLabel` (inglês) é
fallback; chaves `t.module_*` nas traduções são ownership do Antenor (ficheiro
translations.ts deliberadamente não tocado pelo Claude para evitar colisão).
