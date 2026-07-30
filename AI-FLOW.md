# AI-FLOW

A working method for building software with an AI agent.

**Reader:** myself, on the next project.
**What it saves:** relearning the same lessons the hard way.
**Role of AI:** assistant, not author. The agent is hands; I am the one who sets the task.

---

## The feature cycle

The order of steps, and who does each one. Every section below covers one part of this table. Skipping a step is a decision — make it deliberately.

| # | Step | Who | Done when | Section |
|---|---|---|---|---|
| 1 | Read the requirement from the source | me | I can state it without reopening the spec | — |
| 2 | Lay out the options, A/B/C | me, chat as opponent | each option has a mechanism, not a preference | Planning |
| 3 | Choose, and name the cost | me | the cost is written down, not felt | Cross-cutting |
| 4 | Record the decision | me | an ADR exists, or the fork is in the handoff | Templates |
| 5 | Write the prompt | me | Context / Do / Do not, one task, paths from `ls` | Prompt |
| 6 | Write the code | agent | — | — |
| 7 | Review and verify | me | diff read in full, action performed by hand | Accepting the work |
| 8 | Commit and hand off | me | one logical change; the reasoning is recorded | Delivery |

**Steps 1–4 do not involve the agent.** It was not present for the reasoning and cannot reconstruct it.

**How it breaks:** the tempting shortcut is 1 → 5, straight from requirement to prompt. It works, and produces code with no record of why that shape was chosen. Later the code is the only surviving answer to a question nobody can reconstruct.

**Arguments from popularity fail at step 2.** "X is the standard" names no mechanism. Replace it with: what exactly does X include, and is that part needed here.

---

## Cross-cutting

Applies at every step of the cycle, not at one point in it.

### Cost next to benefit
A decision is not made until its cost is named. Benefit and cost are two required fields, not one.

**How it breaks:** the benefit looks forward — what I can do now; the cost looks sideways — where this will break. A cost always exists, even for the right choice. If it is not visible, it was not looked for. Common substitution: naming a difference in the number of steps instead of the point of failure.

### Silence looks like success
Every automated step prints what it actually did: which file, which value, how old.

**How it breaks:** a failure with no symptom goes unnoticed for months. Look wherever a check tests a similar property instead of the real condition — newest file instead of the required file, presence of a button instead of the right to act.

### Show the thing, then ask about its behaviour
Code or an example goes on screen first — questions about consequences come after. A term is explained before its first use, not after its tenth.

**How it breaks:** an abstraction that was never shown does not work. Asking about a thing someone has not seen tests imagination, not understanding — the answer sounds reasonable and means nothing.

### Context is a budget that grows
The model is stateless. What looks like memory is the whole conversation resent on every call — so each call inside a session costs more than the previous one. One instruction to an agent expands into 10–15 calls, each carrying everything before it.

Practical consequences:

- **Expensive model where calls are few and the cost of a mistake is high.** Bulk classification goes to the cheapest model; an architectural decision made once goes to the strongest one.
- **A long session is not free.** Splitting work into sessions and carrying context forward deliberately costs less than one session that grows all day.
- **Hand-written handoff beats automatic compaction.** Compaction decides what survives; a handoff means I decide.
- **Check what the environment bills.** An API key present in the shell environment overrides the subscription and charges per token instead.

**How it breaks:** the cost is invisible per call and only shows up as a limit hit mid-task. Nothing warns you that call 40 costs several times what call 5 cost.

### Trust level is set by cost of error and reversibility
How much the agent decides on its own is a dial, not a setting. It is chosen per task from two questions: what does a mistake here cost, and how easily is it undone.

Working detector: **how long until I notice this is wrong, and how hard is it to roll back?** Minutes and one `git revert` — delegate. Silent data loss discovered next week — review every step.

**How it breaks:** the dial gets set once, on a task where it was safe, and then quietly stays there for a task where it is not.

### A subagent buys a clean context and a tool list it cannot exceed
A subagent is a separate call with its own system prompt and its own tools. It does not see the main conversation — only the task handed to it. Reach for one when either property is the point: a task that must not inherit the framing already built up, or a task that must be *unable* to do something. Removing `Edit` and `Write` is a stronger guarantee than instructing it not to edit.

**How it breaks:** a clean context is also an empty one. Everything the subagent needs has to be in the task text, and whatever is missing it fills in silently — same as any agent. A vague task returns confident output built on assumptions nobody stated.

### Verify the AI's prediction, do not accept it
An agent's prediction about system behaviour is a hypothesis. It is verified by running it, not by agreeing with it.

**How it breaks:** confidence of phrasing does not correlate with correctness; AI speaks about the verified and the guessed in the same tone. My own "I don't see the difference between these options" is more often closer to the truth than someone else's certainty — do not fold it.

---

## Planning

### Plan backwards from the deadline
Not "how much will we get through", but backwards from the moment of delivery. The delivery block is cut out first and declared untouchable.

**How it breaks:** work expands to fill available time. An unreserved delivery slot gets eaten by tasks that feel more important while the deadline is still far away.

### Trial deploy early
On a new project — deploy in the middle of the work, not at the end. On an existing pipeline the rule is unnecessary: the path has already been walked.

**How it breaks:** some configuration exists only in production and is invisible locally by any means — SPA rewrites, environment variables, migrations, database access. It surfaces at the last moment and costs an hour instead of fifteen minutes.

---

## Prompt

### Structure: Context / Do / Do not
Three blocks, imperative form. Context — what the task is. Do — what to do and where to look. Do not — what is categorically forbidden.

**How it breaks:** the prompt exists to cut off deviations. Without a "Do not" block the agent fills gaps at its own discretion, and does it silently — the diff comes back containing things nobody asked for.

```
Context:
  <one task>

Do:
  <actions>
  <where the source is: mockup, tokens, constants>
  <explicit limits: the only permitted state is ...>
  <what may be edited outside the main file>

Do not:
  <prohibitions>
  Do not add files outside the stated context
```

A prompt is an artifact: 2–3 iterations, each closing one specific gap.

### One task per prompt
Two tasks means two prompts.

**How it breaks:** the smaller the task, the better it can be described and controlled. A diff covering two tasks cannot be read as a diff covering one — you accept both or revert both, and the correct half goes out with the incorrect one.

### I define expected behaviour, before the code
For non-trivial logic the agent's first step is to produce a list of cases, `input → expected result`, and stop. I read the list, correct it, approve it — only then code and tests.

**How it breaks:** an agent writing implementation and tests together fits them to each other. The test describes what the code does instead of what the code should do: green, with the bug still in place. Cross-checking does not help here — there is nothing to check against while expectations are written down nowhere.

### Facts go into the prompt only from a source
Paths are copied from `ls` output, not typed from memory. Sizes and colours come from the specification text, not measured off a raster image. Field names, response codes, environment variable names come from the schema or contract, not from recall.

**How it breaks:** memory produces the plausible, a source produces the correct. The agent creates a file at a non-existent path and reports success; builds a query against a field the schema does not have.

---

## Accepting the work

### Work is not done until it is verified
Not "the code is there" — but "the action happens".

- **Frontend:** every interactive element is clicked by hand.
- **Backend:** tests are mandatory, and the logic is additionally exercised by hand. A green test proves only what the test describes.

**How it breaks:** the code compiles, everything looks right, the logic is written — and not wired up. A button without a handler looks exactly like a button with one; an endpoint with the right signature looks exactly like an endpoint that actually does something. Checking with the eye does not replace checking by action.

### Read the whole diff; after an overwrite, `grep` the critical lines
The diff is read in full, including the part after the point where everything already looks correct.

**How it breaks:** an overwrite displays collapsed sections as unchanged without proving it. A line nobody touched and nobody checked quietly disappears.

### Never turn off manual approval
Per-edit approval, always. "Allow all edits" is not enabled — not at the end of the day, not on a small task.

**How it breaks:** the mode is switched off once and applies to every subsequent edit, including the ones you would have rejected.

### The agent adds extras and silently skips instructions
The result is checked against the source — mockup, API contract, database schema — and against the prompt text, point by point.

**How it breaks:** a deviation from instructions comes with no notification. The agent does not write "I decided otherwise" — it simply does otherwise. The prompt is then treated as fulfilled because the code works.

### Cut off tooling detours
A tool that does not write code into the target files is not work on the task. Cut it immediately.

**How it breaks:** the agent drifts into tooling because the task is easier to state there. It looks like progress — commands run, output appears, target files stay unchanged.

---

## Delivery

### Known gaps in the README
Known shortcomings are listed, each with its mechanism. A suppressed warning with a line in the README is a tradeoff; a silent suppression is debt.

**How it breaks:** an unlisted shortcoming reads as unnoticed, a listed one reads as a deliberate tradeoff. The reviewer will find exactly what you found — but without your framing.

---

## Session ritual

- **Fuel check-in, 1–5** at the start: 🟢 full scope · 🟡 minimum only · 🔴 light review. Scope adjusts without guilt.
- **Gates** at the end: 2–3 comprehension questions. The handoff is not written until they are answered.
- **Handoff** — context for the next session.
- **A day counts** only if it produced a tangible artifact.

### Gates go from one word to one sentence
A comprehension check is a ladder. First rung: answerable in one word or a short phrase. Only if that lands, second rung: answerable in one sentence, where the thought has to be assembled. Both correct means it is learned.

**How it breaks:** a first question that already requires the whole concept cannot tell "did not understand the concept" from "did not understand the question". The wrong answer carries no information about where the gap is, and the re-explanation is aimed by guess.

## Templates

`templates/` holds the files this method assumes. Copy them into the new repo and fill them in.

| File | Goes to | Purpose |
|---|---|---|
| `CLAUDE.md` | repo root | the only file the agent reads unprompted — project rules and real structure |
| `handoff.md` | wherever session notes live | continuity between sessions: what was decided and why |
| `adr.md` | wherever decisions live | one architectural decision, with its cost |
| `planner.md` | `~/.claude/agents/` | lays out options for step 2 — no recommendation, cost per option |
| `reviewer.md` | `~/.claude/agents/` | checks a diff against its prompt at step 7, and lists the manual checks |

The two subagents go to **user scope** (`~/.claude/agents/`), not into each project. A per-project copy shadows the global one and then drifts from it; one file seen by every project has no copy to drift.

Each template carries a maintenance comment at the bottom explaining how it fails. Delete the comment in the real file.

**Where they go is not prescribed.** This method does not dictate a folder layout — the templates describe content, the project decides placement.

## Roles

| | Does | Does not |
|---|---|---|
| Me | sets the task, makes decisions, reads the diff | does not accept unread |
| Agent | writes code from the prompt | does not decide architecture |
| Chat | reviews prompts and diffs, acts as opponent | does not author my formulations |
