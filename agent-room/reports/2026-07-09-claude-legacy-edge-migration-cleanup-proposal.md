# Proposta de EPIC — Legacy Edge/Migration Cleanup

Agent: Claude / Fable
Date: 2026-07-09
Status: PROPOSAL — needs architecture decision (Victor/ChatGPT). Nada executado.
Contexto: modo autónomo pós-reconciliação (#29 merged). Este é o drift RESTANTE
(fora do objetivo webhook/send/resiliência, que está 100% reconciliado).

## Porque é uma decisão de arquitetura (não autónoma)
Deprecar funções ou reconciliar o ledger de migrações são ações irreversíveis/
ambíguas: uma função "não invocada por main" pode ter chamador EXTERNO (webhook
de terceiro, integração, onboarding). Remover produção ou reescrever o ledger
exige decisão humana. Por isso paro e proponho.

## Drift 1 — Edge functions legadas (12 candidatas)
20 funções ACTIVE em produção. Núcleo ativo (9) reconciliado (ver KB 19).
Restantes, deployed mas NÃO invocadas por main (frontend/edge/cron):

| Função | Provável estado | Recomendação |
|---|---|---|
| `send-whatsapp` | substituída por `whatsapp-send` | deprecar após confirmar 0 chamadores |
| `evolution-webhook` | substituída por `whatsapp-webhook`; em repo, não deployed | remover do repo |
| `automation-engine` | substituída por `automation-execute` | deprecar |
| `lead-intake` | substituída por `landing-lead` | deprecar |
| `twilio-webhook` | integração Twilio (não usada?) | VERIFICAR chamador externo antes |
| `appointment-api` | API de agendamentos externa? | VERIFICAR chamador externo |
| `setup-org-storage` | setup de storage (onboarding?) | VERIFICAR se corre no signup |
| `sync-whatsapp-history` | sync histórico (one-off?) | VERIFICAR |
| `whatsapp-sync` | sync (one-off?) | VERIFICAR |
| `debug-api`, `debug-whatsapp`, `diagnostic` | debug/diagnóstico | remover de produção (não devem estar ACTIVE) |

Processo recomendado por função: (1) confirmar 0 chamadores (logs/grep externo);
(2) se debug/legado → remover da produção (gate deploy) e do repo; (3) se tem
chamador externo → committar a fonte a main (reconciliar) + documentar.

## Drift 2 — Ledger de migrações (54 aplicadas vs 34 ficheiros)
Schema CONSISTENTE (tudo aplicado); só o ledger de nomes está desalinhado.
34 migrações aplicadas sem ficheiro homónimo em main (early: core_saas_schema,
whatsapp_saas_v2, payments_rebuild, automation_schema_v2, fix_rls, harden_security,
messages_schema, conversations_schema, …). ~10 ficheiros em main com nomes que
não batem com aplicadas.

Opções:
- **A (recomendada):** criar UM baseline squashed (`00000000_baseline.sql`) via
  `pg_dump --schema-only` do estado atual, e marcar o histórico anterior como
  baseline. Ambientes novos passam a reproduzir prod a partir do baseline.
- **B:** recuperar/committar cada ficheiro em falta 1:1 (trabalhoso; a fonte de
  muitos early não existe — teria de vir de pg_dump na mesma).
- **C:** não fazer nada (aceitar que só se cria ambiente novo via pg_dump).

Risco atual: baixo/não-urgente (nenhum ambiente novo em vista). Só relevante para
onboarding de dev / disaster recovery.

## Impacto se não feito
- Funções legadas ACTIVE: superfície de ataque + confusão (ex.: `debug-*`,
  `diagnostic` expostas). **Prioridade de segurança** rever as debug/diagnostic.
- Ledger: recriar ambiente a partir de main seria incompleto.

## Recomendação
1. **Segurança primeiro:** verificar e (se seguro) desativar `debug-api`,
   `debug-whatsapp`, `diagnostic` em produção (gate deploy) — não devem estar
   públicas/ACTIVE.
2. Depois, deprecar as substituídas (`send-whatsapp`, `evolution-webhook`,
   `automation-engine`, `lead-intake`).
3. Verificar chamador externo para `twilio-webhook`, `appointment-api`,
   `setup-org-storage`, `*sync*` antes de qualquer ação.
4. Ledger: opção A (baseline squashed) quando houver necessidade de ambiente novo.

Tudo isto requer gates (deploy/remoção) → decisão de Victor/ChatGPT.

## Gates
Nada executado. Docs de plataforma atualizados em Produto PR #32.
