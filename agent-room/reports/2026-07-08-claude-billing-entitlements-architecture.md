# Billing Entitlements + Seat Limits — arquitetura (sem Stripe)

Task: NEXT.md Claude tarefas 6, 7, 8
Agent: Claude / Fable
Status: needs_review (arquitetura + SQL draft; nada aplicado, Stripe não tocado, Payments produção intocado)

## 1. Plan keys e módulos (tarefa 6) — proposta para decisão de Victor

Ids de plano estáveis em inglês; preços/nomes comerciais são display e podem
mudar sem tocar nos ids.

| plan key | módulos (entitlements) | team_seats | notas |
|---|---|---|---|
| `founder` | todos os active | ilimitado (null) | plano interno/legacy — o default de quem já usa hoje |
| `solo` | dashboard, crm, calendar, messages, finance, settings, billing | 1 | tatuador individual |
| `studio` | solo + team, automations | 5 | estúdio pequeno |
| `pro` | todos os active (inclui agents; reports quando existir) | 15 | topo |

Regras fixas independentes de plano: `dashboard`, `settings`, `billing` nunca
são removidos (core). Valores comerciais (nomes, preços, seats exactos) são
DECISÃO DE VICTOR — a tabela acima é proposta técnica de partida.

## 2. Schema (draft, não aplicado)

```sql
CREATE TABLE public.plans (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  key text UNIQUE NOT NULL,              -- 'founder' | 'solo' | 'studio' | 'pro'
  name text NOT NULL,                    -- display
  entitlements jsonb NOT NULL DEFAULT '{}'::jsonb,
  -- shape: { "modules": ModuleId[], "limits": { "team_seats": int | null } }
  is_active boolean NOT NULL DEFAULT true,
  created_at timestamptz NOT NULL DEFAULT now()
);
-- RLS: SELECT para authenticated (catálogo público interno); escrita só service_role.

ALTER TABLE public.organizations
  ADD COLUMN IF NOT EXISTS plan_id uuid REFERENCES public.plans(id);
-- plan_id NULL = 'founder' (default-tudo) → ZERO mudança de comportamento no dia 1.
```

`billing_subscriptions` (provider, período, status) fica para a slice Stripe —
YAGNI até existir provider. O enforcement depende só de `organizations.plan_id`.

## 3. Seat limits DB-side (tarefa 7) — draft

Ponto único de enforcement no INSERT de convites (não confiar na UI):

```sql
CREATE OR REPLACE FUNCTION public.enforce_team_seat_limit()
RETURNS trigger AS $$
DECLARE
  seat_limit int;
  seats_used int;
BEGIN
  SELECT (p.entitlements->'limits'->>'team_seats')::int INTO seat_limit
  FROM public.organizations o
  LEFT JOIN public.plans p ON p.id = o.plan_id
  WHERE o.id = NEW.organization_id;

  IF seat_limit IS NULL THEN RETURN NEW; END IF;  -- sem plano/ilimitado

  SELECT (SELECT count(*) FROM public.users u
           WHERE u.organization_id = NEW.organization_id)
       + (SELECT count(*) FROM public.invitations i
           WHERE i.organization_id = NEW.organization_id
             AND i.status = 'pending' AND i.expires_at > now())
  INTO seats_used;

  IF seats_used >= seat_limit THEN
    RAISE EXCEPTION 'seat_limit_reached';  -- código estável para a UI (Antenor)
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER SET search_path = public;

CREATE TRIGGER enforce_team_seat_limit_trg
  BEFORE INSERT ON public.invitations
  FOR EACH ROW EXECUTE FUNCTION public.enforce_team_seat_limit();
```

Quando a slice soft-remove aterrar, a contagem de users passa a filtrar
`status='active'` (revogados não ocupam seat).

## 4. Resolução de acesso efectivo (tarefa 8)

Fórmula única, partilhada entre Login enforcement e sidebar:

```
plan_modules  = plan_id NULL ? todos_os_active : plans.entitlements.modules
admin         → plan_modules (+ billing sempre)
não-admin     → normalizeModuleIds(users.permissions) ∩ plan_modules
```

Frontend: hook `useOrgEntitlements` (padrão CLAUDE.md: org-scoped, bloqueia
sem organizationId, `{data,isLoading,error}`). Fallback em erro: NEGAR módulos
não-core com erro visível — nunca conceder tudo em falha.

Downgrade: módulos fora do plano ficam inacessíveis; `users.permissions` nunca
é apagado (reversível no re-upgrade). Nenhum dado é deletado por downgrade.

Stripe (fora de âmbito): webhook de subscrição escreve `organizations.plan_id`.
Todo o enforcement já está atrás do plan_id — o provider vira só fonte de escrita.

## 5. Ordem de implementação proposta

1. Migração plans + plan_id + seed founder (NULL-safe) — gate de apply.
2. `useOrgEntitlements` + estados UI do Antenor (usa o erro `seat_limit_reached`).
3. Trigger de seats — gate de apply.
4. Login enforcement (depois de PR #13 merged + migrações aplicadas).
5. Stripe — EPIC próprio, mais tarde.

## Decisões pedidas

1. Valores comerciais dos planos (secção 1).
2. Aprovar `plan_id NULL = founder/default-tudo` como estratégia de rollout.
