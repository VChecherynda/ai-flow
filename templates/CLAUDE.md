# <PROJECT NAME>

<One line: what this is, what stack, why it exists.>

## Commands
- Package manager: **<pnpm | npm | yarn>** (state which, so the agent does not guess)
- Dev: `<cmd>` · Build: `<cmd>` · Test: `<cmd>`

## Working with me
- Respond in <language>.
- Plan first, then code. Use plan mode for new features.
- Never push. Prepare the commit; I run `git push` myself.
- Do not create files I did not ask for (READMEs, examples, "just in case" configs).
- For non-trivial logic: list the cases `input → expected result` first and stop. Wait for approval.

## Architecture
- <Real folder layout, copied from `ls`, not from intent.>
- <Where business rules live.>
- <What does not exist: no backend / no SSR / no database — say it explicitly.>

## Decisions
Architectural decisions live in `<path>`. Check there before changing an established approach.

---

<!--
MAINTENANCE — delete this comment in the real file.

This file is the only source the agent reads without being asked.
A wrong line here is not a typo: it is the agent being told, with authority,
to do the wrong thing — and it will look confirmed.

Two failure modes seen in practice:

1. Written once at kickoff, never re-checked. The stack changed, the file did not.
2. Written as intent ("we will use DDD") and read as description.
   Say what IS. Put targets in an ADR, not here.

Re-check against disk whenever the structure moves:
  - every path in this file must exist in `ls`
  - every technology named must exist in the dependency manifest
-->
