# INSTALL — AI-executable install (給 AI 自動安裝用)

> **You are an AI assistant in the user's Claude Code session and they pasted the `goal-workflow-designer` repo link.** Follow this file to install both skills into the right place. Show the user a diff/summary before writing any file.
>
> **你是使用者 Claude Code session 裡的 AI，使用者貼了本 repo 連結。** 照此檔把兩支 skill 裝到正確位置。寫任何檔前先給使用者看 diff/摘要。

These are **user-level skills** — they live under `~/.claude/skills/` and work across all your Claude Code projects. (`~` = your home dir, e.g. `C:\Users\<you>` on Windows, `/home/<you>` on Linux/macOS.)

## What gets installed

Two sibling task-shaping skills:

- **`goal`** — designs a precise, executable `/goal` prompt (depth: one agent + rubric, iterate to a quality bar).
- **`workflow-shaper`** — shapes a task into a runnable Dynamic Workflow (breadth: many agents in parallel, cross-checked), with a worth-it gate up front.

> ⚠️ Neither skill *is* the `/goal` feature or the Workflow feature. They are **design coaches** that produce the prompt/spec you feed to those features. You do **not** need them installed to use `/goal` — they just make the prompt that goes into it far more precise.

## Steps (AI: execute in order, show diffs first)

1. **Fetch the two skill files** (raw URLs, `main` branch):
   - `https://raw.githubusercontent.com/dragon375014/goal-workflow-designer/main/skills/goal/SKILL.md`
   - `https://raw.githubusercontent.com/dragon375014/goal-workflow-designer/main/skills/workflow-shaper/SKILL.md`

   Use WebFetch, or have the user run the `git clone` fallback below if web fetch is unavailable.

2. **Write them to the user-level skills dir** (create folders if missing — **flat, one folder per skill, never a category subfolder**; Claude Code's registry only reads the first level):
   - `~/.claude/skills/goal/SKILL.md`
   - `~/.claude/skills/workflow-shaper/SKILL.md`

   If a file already exists, **show the user a diff and ask before overwriting** (they may have local edits).

3. **Append the triggers to `~/.claude/CLAUDE.md`** (create the file if missing). Only add a block that isn't already present:

   ```markdown
   # goal
   - **goal** (`~/.claude/skills/goal/SKILL.md`) — Goal prompt design coach (depth). Trigger: `/goal`
   When the user types `/goal`, OR uses natural language indicating they want to design a goal (e.g. "I want to design a goal", "幫我寫個 goal prompt"), invoke the Skill tool with `skill: "goal"` before doing anything else.

   # workflow-shaper
   - **workflow-shaper** (`~/.claude/skills/workflow-shaper/SKILL.md`) — Shape a task into a runnable Dynamic Workflow (breadth), worth-it gate first. Trigger: `workflow-shaper`
   When the user wants to run the same check/change across many independent units, says "use a workflow", or asks whether a task is worth a workflow, invoke the Skill tool with `skill: "workflow-shaper"`.
   ```

4. **Verify**: confirm both `SKILL.md` files exist at the paths above and that the two trigger blocks are present in `~/.claude/CLAUDE.md`. Report the result to the user.

5. **Tell the user how to use it**:
   - `/goal <one-line task>` to design a depth goal.
   - Describe a many-unit task (or say "use a workflow") to trigger `workflow-shaper`.
   - Output goals are saved under `~/.claude/goals/<name>/`.

## Manual fallback (no AI / no web fetch)

```bash
# Clone, then copy each skill folder into ~/.claude/skills/
git clone https://github.com/dragon375014/goal-workflow-designer.git /tmp/gwd
cp -r /tmp/gwd/skills/goal ~/.claude/skills/goal
cp -r /tmp/gwd/skills/workflow-shaper ~/.claude/skills/workflow-shaper
```

Then paste the two trigger blocks from Step 3 into your `~/.claude/CLAUDE.md`.

## Companion repo

If you also want **structural governance** (architecture-completeness gate, cross-layer trace-lock, defensive linters), pair this with [`claude-skills-governance-meta`](https://github.com/dragon375014/claude-skills-governance-meta). This repo shapes *what to do* (the task spec); that repo guards *how it's built* (the structure). See the README's "Companion repos (姐妹 repo)" section. You can also install the whole five-repo toolchain in one command: `npx specmit init`.
