# START Protocol

When Victor writes `START` to an agent, the agent must:

1. Read `rtzforyou/keyros-engine/CURRENT_ORCHESTRATION.md`.
2. Read `rtzforyou/keyros-engine/OWNERSHIP.md`.
3. Read its own inbox:
   - Claude/Fable: `agent-room/inbox/claude-current.md`
   - Antenor: `agent-room/inbox/antenor-current.md`
4. Work by EPIC, not by isolated micro-tasks.
5. Execute all Sprints inside the current EPIC automatically.
6. Update reports in `agent-room/reports/`.
7. Create branches, commits and PR Drafts when code changes.
8. Run build/typecheck when relevant.
9. Stop only at hard gates:
   - Supabase migration apply
   - production data change
   - main merge
   - production deploy
   - RLS/Auth/security boundary change
   - cross-agent conflict
   - architectural decision
   - technical block

Final response must be short:

```text
EPIC:
Status:
Sprints executed:
Branches:
Commits:
PRs:
Reports:
Blocks:
Next recommended EPIC:
```

Do not paste full reports into chat.
