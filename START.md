# START Protocol

When Victor writes `START` to an agent, the agent must:

1. Read `rtzforyou/keyros-engine/OWNERSHIP.md`.
2. Read its own inbox:
   - Claude/Fable: `agent-room/inbox/claude-current.md`
   - Antenor: `agent-room/inbox/antenor-current.md`
3. Work by EPIC, not by isolated micro-tasks.
4. Execute all Sprints inside the current EPIC automatically.
5. Update reports in `agent-room/reports/`.
6. Create branches, commits and PR Drafts when code changes.
7. Run build/typecheck when relevant.
8. Stop only at hard gates:
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
