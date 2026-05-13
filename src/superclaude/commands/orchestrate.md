---
name: orchestrate
description: "Single-entry orchestration with staged optimization: explore, evaluate, select strategy, execute, and monitor drift. Uses Agent Teams when scope is large."
category: orchestration
complexity: high
mcp-servers: [serena]
personas: []
---

# /sc:orchestrate - Staged Optimization Orchestrator

**Single command, five internal stages.** You MUST run stages 1–5 in order. Do not skip or merge stages.

Reference: [Claude Code Agent Teams (ja)](https://code.claude.com/docs/ja/agent-teams). Optional intro: [YouTube](https://www.youtube.com/watch?v=zm-BBZIAJ0c).

## Usage

```
/sc:orchestrate [task-description]
```

## MANDATORY FIVE STAGES (run in order)

### Stage 1: Task classification and scope estimation

- Produce a **lightweight plan only**: extract impact scope (affected files, domains, dependencies). Do NOT write implementation steps.
- Classify scope as **small** | **medium** | **large**:
  - small: few files, single domain, minimal dependencies
  - medium: multiple areas, some parallelization benefit, moderate dependencies
  - large: multi-domain, clear benefit from multiple agents or worktrees, many dependencies
- Persist the plan: `write_memory("orchestrate_plan", "<concise plan summary and scope>")`.
- Output to user: scope label and one-paragraph impact summary.

### Stage 2: Strategy candidate enumeration

- List exactly three candidates (do not add others):
  1. **Single session** – one context, no parallelism.
  2. **In-session parallel** – same session, parallel tool calls and TodoWrite-split work (Wave → Checkpoint → Wave).
  3. **Agent Team** – multiple Claude Code instances coordinated via [Agent Teams](https://code.claude.com/docs/ja/agent-teams) (requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`).
- Describe each in one line. Do not choose yet.

### Stage 3: Scoring and automatic selection

- Score the three candidates on: **cost**, **reproducibility**, **future extensibility** (brief qualitative reasoning only; no numeric formula).
- **Automatically select exactly one** strategy. Do not ask the user to choose.
- Output: selected strategy and 1–2 sentence justification.
- Persist: `write_memory("orchestrate_strategy", "<selected strategy and short reason>")`.

### Stage 4: Execution

- Branch by **selected strategy**:

  - **Single session**: Proceed with implementation in this session. Delegate to `/sc:implement` or `/sc:task` as needed. No parallel structure required.
  - **In-session parallel**: Use TodoWrite to split work; use parallel Read/Edit waves where independent; then checkpoint and continue. Delegate to `/sc:implement` or `/sc:task` with parallel patterns.
  - **Agent Team**:
    1. Check that Agent Teams are enabled: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in environment or in `.claude/settings.json` under `"env"`. If not set, tell the user to set it and point to [Agent Teams docs](https://code.claude.com/docs/ja/agent-teams).
    2. Create the team via natural language. Example prompt to use (adapt placeholders):  
       "[Task summary]. Create an agent team for this. Spawn N teammates with roles: [list roles, e.g. backend, frontend, tests]. Require plan approval before they make any changes."
    3. Follow official docs: shared task list, plan approval for members, and **clean up the team** when done (always use the leader to clean up; do not let teammates run cleanup).
    4. Document known limitations: session resume does not restore in-process teammates; task status may lag; one team per session; no nested teams.

- After execution, output a short checkpoint summary (what was done, what remains if any).

### Stage 5: Drift monitoring

- **Whenever the user indicates a direction change** (e.g. "やり直し", "こっちの方がいい", "別の方法で", "やっぱり違う方針で"), before proceeding you MUST:
  1. Call `read_memory("orchestrate_plan")` and optionally `think_about_task_adherence` to assess alignment with the current task.
  2. Compare the **new direction** to the stored plan: list overlapping work and affected scope.
  3. Output a short **drift summary** and **rework risk** (what may be duplicated or invalidated).
  4. Ask explicitly: **「元の計画と変わっています。重複リスク: [X]。続行しますか？」** (or equivalent).
  5. Only after user confirmation, proceed. Optionally update `orchestrate_plan` with the new direction if they confirm.

- During normal execution (no direction change), continue without this block.

## Serena MCP

- Use `write_memory` / `read_memory` for `orchestrate_plan` and `orchestrate_strategy`.
- If available, use `think_about_task_adherence` when evaluating drift in Stage 5.

## Boundaries

**Will:**
- Run the five stages in order and persist plan/strategy in memory.
- Automatically select one strategy (single session, in-session parallel, or Agent Team) and execute accordingly.
- Enforce drift check and confirmation on any user-expressed direction change.

**Will Not:**
- Ask the user to choose among strategies; selection is automatic in Stage 3.
- Skip Stage 5 when the user says they want to redo or change approach; always show diff and rework risk and ask for confirmation.

## Examples

### Single-session outcome (small scope)

```
/sc:orchestrate "add dark mode toggle to settings screen"
# Stage 1: scope small (one UI area, few files)
# Stage 2–3: single session selected
# Stage 4: implement in this session
```

### In-session parallel (medium scope)

```
/sc:orchestrate "refactor auth module and add unit tests"
# Stage 1: medium (auth + tests, parallelizable)
# Stage 2–3: in-session parallel selected
# Stage 4: TodoWrite + parallel waves, then checkpoint
```

### Agent Team (large scope)

```
/sc:orchestrate "implement checkout flow: API, UI, and E2E tests"
# Stage 1: large (backend, frontend, test domains)
# Stage 2–3: Agent Team selected
# Stage 4: verify CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1, then create team with plan approval, shared tasks, cleanup when done
```

## Note on Agent Teams

Agent Teams are experimental. Limitations (see [docs](https://code.claude.com/docs/ja/agent-teams#limitations)): in-process teammates are not restored on session resume; task status may lag; shutdown can be slow; one team per session; no nested teams; leader is fixed. Always clean up via the leader.
