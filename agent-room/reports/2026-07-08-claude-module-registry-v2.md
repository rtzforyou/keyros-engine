# Canonical Module Registry v2 — modelo de domínios aprovado

Task: EPIC Canonical Module Registry (NEXT.md 2026-07-08 16:13 + MASTER_STATE.md)
Agent: Claude / Fable
Status: needs_review — SUPERSEDE a proposta v1 (2026-07-08-claude-module-registry-proposal.md)
Decisões do ChatGPT incorporadas: manter `crm`; contacts/deals/pipeline/activities dentro de `crm`; `finance` em vez de `expenses`; `billing` para planos/seats/subscrições/faturas/provider; `agents` em vez de `ai`; bounded contexts, não ecrãs.

## 1. Registry v2 — 11 domínios

| id | labelKey | status | teamAssignable | planControlled | dependsOn |
|---|---|---|---|---|---|
| `dashboard` | module_dashboard | active | sim | não (core) | — |
| `crm` | module_crm | active | sim | sim | — |
| `calendar` | module_calendar | active | sim | sim | — |
| `messages` | module_messages | active | sim | sim | — |
| `automations` | module_automations | active | sim | sim | messages |
| `team` | module_team | active | sim (aviso) | sim (via seats) | — |
| `billing` | module_billing | active | **não** (admin-only) | não (core) | — |
| `finance` | module_finance | active | sim | sim | — |
| `reports` | module_reports | **reserved** | não (ainda) | sim | dashboard |
| `settings` | module_settings | active | sim (aviso) | não (core) | — |
| `agents` | module_agents | active | sim | sim | messages |

Racional das excepções:
- `billing` não é atribuível a membros: gere plano/seats/pagamento da própria subscrição — poder de dono. Também é core (nunca desligado por plano): uma org tem de conseguir chegar ao billing para fazer upgrade/regularizar.
- `dashboard`/`settings` core: org paga entra sempre e configura sempre.
- `reports` reservado até existir superfície própria (KPIs hoje vivem no dashboard).
- `team`/`settings` atribuíveis com aviso na UI (dão poder de configuração a staff; admin já tem tudo por role).
- `dependsOn` é soft (UI sugere/avisa; enforcement não bloqueia).

## 2. Aliases legados (leitura; nunca escritos de novo)

```
contacts  → crm
deals     → crm
pipeline  → crm
payments  → finance
expenses  → finance
aiAgent   → agents
ai        → agents
```

Consequência de granularidade (aceite pelo modelo de bounded contexts): deixa de ser possível dar `contacts` sem `deals` — ambos são `crm`. O convite existente em produção (`[messages, dashboard, pipeline, contacts, calendar]`) normaliza para `[messages, dashboard, crm, calendar]`.

## 3. Shape TypeScript (implementação futura em `lib/moduleRegistry.ts`)

```ts
export type ModuleId = 'dashboard' | 'crm' | 'calendar' | 'messages' | 'automations'
  | 'team' | 'billing' | 'finance' | 'reports' | 'settings' | 'agents';

export interface ModuleDefinition {
  id: ModuleId;
  labelKey: keyof Translations;      // display separado do id (t.module_*, 3 idiomas)
  status: 'active' | 'reserved';
  teamAssignable: boolean;
  planControlled: boolean;
  dependsOn?: ModuleId[];
}

export const MODULE_REGISTRY: ModuleDefinition[] = [/* tabela da secção 1 */];
export const LEGACY_MODULE_ALIASES: Record<string, ModuleId> = {/* secção 2 */};
export const normalizeModuleIds = (raw: string[]): ModuleId[] =>
  [...new Set(raw.map(id => LEGACY_MODULE_ALIASES[id] ?? id))]
    .filter(isRegisteredModuleId);
```

Routing interno do app (tabs contacts/pipeline/payments/expenses…) NÃO muda: várias tabs podem pertencer ao mesmo domínio (`crm` gateia contacts+pipeline; `finance` gateia payments+expenses). O registry gateia acesso por domínio; a navegação continua por ecrã.

## 4. Plano — Team Permissions Editing (v2, do registry)

1. Migração de dados (pequena, 1 convite): normalizar `invitations.permissions` com os aliases da secção 2. `users.permissions` está limpo (`[]` em 7/7).
2. `lib/moduleRegistry.ts` + chaves `module_*` em `lib/translations.ts` (en/pt/fr).
3. TeamContent: checkboxes do convite passam a domínios do registry (`teamAssignable && status='active'`), com aviso em `team`/`settings`; leitura com `normalizeModuleIds`.
4. `useTeamMembers.updateTeamMember(id, { role, permissions })` — valida ids contra o registry; acção visível só para admin. Defesa: UI + RLS admin-update + trigger guard (produção).
5. Restaurar dialog "Editar Acesso" (JSX no histórico do PR #12) ligado ao hook.
6. Auditoria: INSERT em `public.audit_logs` (existente, RLS admin-only) — `team.role_changed`/`team.permissions_changed`, details `{old,new}`.
7. Last-admin protection (migração): estender `protect_sensitive_user_columns()` para bloquear despromoção do último admin da org.
8. Testes: bateria do guard + edição via UI + multi-tenant + refresh.

Gates: passos 1 e 7 (migrações) exigem autorização de apply; restantes são branch/PR draft dentro da autonomia.

## 5. Plano — Billing: plans, seats, subscriptions (sem Stripe)

1. Migração: tabela `plans` (`id uuid`, `key text unique` ex. 'free'/'pro', `name`, `entitlements jsonb` = `{"modules": ModuleId[], "limits": {"team_seats": int}}`, `is_active`) + `organizations.plan_id uuid NULL REFERENCES plans`. `NULL` = default-tudo → **zero mudança de comportamento no dia 1**.
2. Estrutura preparada para provider: tabela `billing_subscriptions` (org_id, plan_id, status, provider, provider_subscription_id, current_period_end) — criada já vazia ou adiada para a slice Stripe; recomendo adiar (YAGNI até haver provider).
3. `useOrgEntitlements` hook (org-scoped, padrão CLAUDE.md): `{ modules, limits, isLoading, error }`.
4. Acesso efectivo (fórmula única, partilhada com o Login enforcement):
   - admin → `plan.modules`
   - não-admin → `normalizeModuleIds(user.permissions) ∩ plan.modules`
   - `billing`: sempre acessível a admin, independente do plano.
5. Seat limits — enforcement DB-side num ponto único: trigger BEFORE INSERT em `invitations` valida `(membros activos + convites pending) < limits.team_seats`; erro padronizado `seat_limit_reached` para a UI do Antenor (invite blocked / upgrade CTA).
6. Downgrade: módulos fora do plano ficam inacessíveis; `permissions` nunca são apagadas (reversível). Nunca deletar dados por downgrade.
7. Stripe (fora deste plano): webhook subscription → actualiza `organizations.plan_id`/`billing_subscriptions`. Todo o enforcement já está atrás do plan_id — Stripe vira só uma fonte de escrita.

## 6. Decisões pedidas

1. Aprovar v2 do registry (tabela secção 1) — em particular `billing` não-atribuível e `reports` reserved.
2. Valores de negócio dos planos (keys, módulos, team_seats) para o seed.
3. Autorização para implementar a slice Team Permissions Editing (secção 4) como branch/PR draft.
