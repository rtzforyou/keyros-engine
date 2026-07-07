# CLAUDE.md

Instruções permanentes para Claude Code e outros agentes trabalhando no ecossistema Keyros.

## Identidade do projeto

Keyros é o nome atual do ecossistema de produto. O foco é construir uma plataforma de gestão para operações comerciais com CRM, WhatsApp, automações, dashboard financeiro e integrações.

## Direção técnica confirmada

- Projeto com foco multi-tenant.
- Supabase é parte central da stack atual.
- WhatsApp usa Evolution API e Edge Functions do Supabase.
- O sistema precisa evitar lógica presa apenas ao WhatsApp pessoal do Victor.
- Deals, contatos, conversas e automações devem funcionar como produto comercializável.

## Prioridades de implementação

1. Segurança.
2. Rate limit.
3. Idempotência.
4. Retry controlado.
5. Circuit breaker.
6. Logs estruturados.
7. RLS revisado.
8. Testes mínimos antes de deploy.

## Regras de conduta

- Não assumir que dados pessoais do Victor representam regra global do produto.
- Não criar automações baseadas em nomes fixos de grupos ou contatos.
- Não inventar campos de banco sem verificar schema real.
- Não ignorar risco multi-tenant.
- Não tratar integração externa como confiável sem validação.
- Não resolver bug criando exceção permanente sem registrar decisão.

## Formato esperado de resposta técnica

Toda análise relevante deve conter:

1. Problema.
2. Objetivo.
3. O que está confirmado.
4. O que precisa ser verificado.
5. Riscos.
6. Recomendação.
7. Plano de execução.
8. Testes mínimos.

## Regra final

Se a solução aumenta complexidade sem proteger segurança, clareza ou escalabilidade, ela deve ser recusada ou simplificada.
