# UI/UX Plan: Canonical Module Registry Integration — Keyros CRM

This document details the frontend implementation plan for the **Canonical Module Registry** within the Keyros CRM application interface, aligning permissions checklist, module branding, billing entitlements, and upgrade triggers.

---

## 1. Localized Module Registry & Labels

To decouple backend identifiers from localized user experience interfaces, the registry defines canonical system names mapped to human-readable strings (defaulting to Portuguese/PT):

| Canonical ID | Localized Label (PT) | Description / Sub-label | Included in Plans | Team Assignable |
| :--- | :--- | :--- | :--- | :--- |
| `dashboard` | **Painel Principal** | Métricas gerais e performance do estúdio | Free, Premium, Enterprise | Yes |
| `crm` | **CRM / Clientes** | Contactos, pipeline de vendas e negócios | Free, Premium, Enterprise | Yes |
| `calendar` | **Agenda** | Marcações de sessões e horários | Free, Premium, Enterprise | Yes |
| `messages` | **Mensagens** | Integração WhatsApp e chat integrado | Free, Premium, Enterprise | Yes |
| `automations`| **Automações** | Triggers automáticos e respostas rápidas | Premium, Enterprise | Yes |
| `team` | **Equipa** | Gestão de acessos e membros | Free, Premium, Enterprise | No (Admin only) |
| `billing` | **Faturamento** | Gestão de subscrições, planos e faturas | Free, Premium, Enterprise | No (Admin only) |
| `finance` | **Financeiro** | Controle de receitas e despesas do estúdio | Premium, Enterprise | Yes |
| `reports` | **Relatórios** | Relatórios de faturação e KPIs | Enterprise | Yes |
| `settings` | **Definições** | Configurações gerais do estúdio | Free, Premium, Enterprise | No (Admin only) |
| `agents` | **Assistente IA** | Mensagens sugeridas e IA generativa | Enterprise | Yes |

---

## 2. Permissions Checklist (Admin Member Editing)

When managing a team member's access, the owner/admin sees a checkbox grid generated dynamically from the assignable list of modules:

### UI Interface Layout Mockup
```text
+-----------------------------------------------------------------+
| Editar Permissões de Acesso: [Marcos Silva]                     |
|                                                                 |
| Selecione os módulos que este membro da equipa pode aceder:     |
|                                                                 |
| [X] Painel Principal (dashboard)                                |
| [X] CRM / Clientes (crm)                                        |
| [X] Agenda (calendar)                                           |
| [ ] Mensagens WhatsApp (messages)                               |
| [ ] Controle Financeiro (finance)         [PRO] <- locked badge |
| [ ] Automações de Clientes (automations)  [PRO]                 |
| [ ] Assistente IA (agents)                [VIP] <- locked badge |
|                                                                 |
| [ Cancelar ]                                    [ Gravar ]      |
+-----------------------------------------------------------------+
```

### UX Behavior rules
1. **Interactive Checkboxes:** Only modules allowed by the *studio's current plan* are interactive.
2. **Entitlement Locks:** Modules not included in the studio's active plan render with a grey lock icon and a gold badge (e.g. `[PRO]` or `[VIP]`). Checking them is disabled. Clicking them triggers the Upgrade modal.

---

## 3. Plan-based Module Locks & Badges

If a module is not included in the studio's active subscription:

- **Sidebar Navigation:** Sidebar links for locked modules (e.g., `Automações` for Free plan, `Assistente IA` for Free/Premium) are rendered with a subtle padlock icon next to the label.
- **Visual State:** The navigation link text is slightly dimmed (opacity: 60%) to signify a premium feature while maintaining discovery.
- **No White Screen:** Direct navigation via manual URL parameters to a locked route is intercepted by a middleware router hook, displaying a premium landing state rather than rendering an empty page or throwing an error.

---

## 4. Upgrade Flow & Modal Prompt

When a user clicks on a locked module from the sidebar, settings list, or permissions checklist:

### Upgrade Prompt Modal UI
```text
+-----------------------------------------------------------------+
|             ✨ Desbloqueie o Módulo: Automações ✨               |
|                                                                 |
| Este recurso está disponível nos planos Premium e superiores.    |
|                                                                 |
| Benefícios do módulo:                                           |
| - Disparo automático de confirmação de marcação via WhatsApp.   |
| - Respostas rápidas automáticas baseadas em status.             |
| - Economia de até 4 horas por semana de trabalho administrativo.|
|                                                                 |
| [ Escolher Plano Premium - €29/mês ]   [ Cancelar ]             |
+-----------------------------------------------------------------+
```

### Action Handling
- Clicking the action button redirects the user to the **Settings -> Faturamento (Billing)** tab instantly.
- The Billing view highlights the target plan (e.g., Premium) in green to guide the checkout action.
