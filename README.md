# goal-workflow-designer

> Two Claude Code skills that interrogate you until your task is shaped into a **precise, executable spec** — a depth `/goal` prompt or a breadth Dynamic Workflow. A design coach, not the engine.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Skill Format](https://img.shields.io/badge/Claude%20Code-skills-purple)](https://code.claude.com/docs/en/skills)
[![Bilingual](https://img.shields.io/badge/docs-EN%20%2B%20繁中-orange)](skills/goal/SKILL.md)

> 🇹🇼 **繁體中文讀者** → 看白話中文說明：[**README-tw.md**](README-tw.md)（這是什麼、裝了有什麼好處、怎麼用，全中文）

**This is a *designer*, not a feature.** It does **not** replace — and is **not required to use** — Claude Code's `/goal` or its Dynamic Workflows. These skills are **front-end coaches**: they interrogate you and emit the precise prompt/spec you then feed to those features. The hard part was never *running* `/goal` or *a workflow* — it's **writing the spec that goes into them**. That's what this designs.

它是**設計器**，不是功能本身。**不取代、也不需要它才能用** `/goal` 或 Dynamic Workflows。這兩支 skill 是**前端引導器**：反問你、產出精準的 prompt/規格，再由你貼進那些功能。難的從來不是「跑」`/goal` 或 workflow，而是「寫出餵進去的規格」——這就是它設計的東西。

---

## The two skills — depth & breadth (兩支 skill — 深度與廣度)

A vague task can be shaped two ways. This repo ships one skill for each axis:

| Skill | Axis | Shapes your task into… | Reach for it when |
|---|---|---|---|
| [**`goal`**](skills/goal/SKILL.md) | **Depth (深度)** | a precise `/goal` prompt — one agent + a rubric, iterating to a quality bar | **one thing** to get right, deep, and converged |
| [**`workflow-shaper`**](skills/workflow-shaper/SKILL.md) | **Breadth (廣度)** | a runnable Dynamic Workflow — many fresh-context agents in parallel, cross-checked | the **same check/change across many** independent units |

**One-second test for which to use: count how many units.** One thing to perfect → `/goal`. The same check across many things → `workflow-shaper`. Many *different* things with a dependency order → that's spec decomposition, see the companion repo [spec-sonar](#companion-repos) below. They compose — a workflow is the for-loop, a goal is the loop body.

判斷用哪個的一秒法則：**數有幾件**。一件做到完美 → `/goal`；同一個檢查套到很多件 → `workflow-shaper`；很多件「異質且有依賴順序」→ 那是規格分解，見下方姐妹 repo spec-sonar。兩者可組合（workflow 是 for-loop、goal 是 loop body）。

---

## Why this exists

In 2026, Claude Code, OpenAI Codex, and Manus Agent shipped a `/goal` feature within weeks of each other; Claude Code later added **Dynamic Workflows** (background multi-agent orchestration). Both loop or fan out agents toward a target. Anthropic's research showed that without a precise target, LLMs hit **context anxiety** when their context window fills — they shortcut to a wrap-up and stop, even mid-task.

But these features only work when the spec is precise. Given a vague prompt, the AI invents its own definition of "done" and quits early — or, for a workflow, fans out on the wrong shape and burns tokens. Given a precise one, it iterates for hours / parallelizes cleanly.

The hard part isn't using `/goal` or a workflow. It's **writing the spec that goes into them**. These skills automate that, and refuse to let you off the hook with vague answers.

---

## Installation

### Option A — AI auto-install (recommended)

Paste this into your Claude Code session and the agent installs both skills into the right place:

```
Read https://raw.githubusercontent.com/dragon375014/goal-workflow-designer/main/INSTALL.md

Then install both skills (goal + workflow-shaper) into my user-level ~/.claude/skills/ by following it.
Show me a diff/summary before writing any file, and don't overwrite a SKILL.md I've edited without asking.
```

The agent fetches [INSTALL.md](INSTALL.md), places `skills/goal/` and `skills/workflow-shaper/` under `~/.claude/skills/`, and appends the two triggers to your `~/.claude/CLAUDE.md`.

### Option B — Manual

```bash
git clone https://github.com/dragon375014/goal-workflow-designer.git /tmp/gwd
cp -r /tmp/gwd/skills/goal           ~/.claude/skills/goal
cp -r /tmp/gwd/skills/workflow-shaper ~/.claude/skills/workflow-shaper
```

Then add the two trigger blocks (see [INSTALL.md](INSTALL.md) Step 3) to `~/.claude/CLAUDE.md`. These are **user-level** skills — they work across all your Claude Code projects. Output goals are saved under `~/.claude/goals/<name>/`.

> ⏫ Upgrading from `skill-to-goal` (the old single-skill repo name)? The repo was renamed and restructured: the goal skill moved from the repo root to `skills/goal/`, and `workflow-shaper` was added. Re-run Option A, or `git pull` and re-copy.

---

## Usage

### `/goal` — depth

```
/goal <one-line task description>     # full Q&A from scratch
/goal --use <name>                    # load a saved goal
/goal --resume <draft-name>           # resume an interrupted Q&A
```

It walks you through five elements (outcome, verification, constraint, iteration policy, error handling), plus a six-step rubric SOP for subjective work, and outputs a goal prompt that keeps the agent iterating for hours instead of quitting in three minutes. Natural language triggers too (`幫我寫個 goal prompt`).

> ⚠️ `/goal` with no arguments is intentionally unsupported — vague triggers produce vague Q&A.

### `workflow-shaper` — breadth

Describe a many-unit task, or say "use a workflow." It runs a **WORTH-IT gate** (5 criteria → green/yellow/red) before anything else:

- 🟢 green → recommends a fan-out shape (parallel / pipeline / map-reduce) and emits a paste-ready `workflow ...` prompt + a draft meta block you can save as a `/<name>` command.
- 🟡 yellow → offers a middle path (split into short workflows / inline-run one unit first).
- 🔴 red (single unit, needs intermediate context, needs mid-run sign-off) → **refuses to emit a workflow prompt** and routes you back to a plain skill or `/goal`.

It never runs the workflow — it shapes the spec and hands it back. See [skills/workflow-shaper/SKILL.md](skills/workflow-shaper/SKILL.md).

---

## What makes the `/goal` skill different from generic prompt skills

A handful of Claude Code skills do prompt design ([prompt-architect](https://github.com/ckelsoe/prompt-architect), [prompt-improver](https://github.com/ndpvt-web/prompt-improver), [prompt-master](https://github.com/nidhinjs/prompt-master)). They are general-purpose. This one is laser-focused on the `/goal` execution model:

| Feature | this repo | Generic prompt skills |
|---|---|---|
| Targets `/goal`'s five-element framework | ✅ | ❌ (general frameworks like CO-STAR, RISEN) |
| Builds a rubric for subjective tasks | ✅ (6-step SOP) | ❌ |
| Context window health gate | ✅ (Phase 0a) | ❌ |
| **Goal-eligibility check (3-light)** | ✅ (Phase 0c) | ❌ |
| KICK-OFF self-cue (agent actually starts, no "continue" needed) | ✅ | ❌ |
| Handoff packaging for context-strained sessions | ✅ (`--resume`) | ❌ |
| Codebase scan via isolated subagent | ✅ | ❌ |
| **A breadth counterpart for many-unit tasks** | ✅ (`workflow-shaper`) | ❌ |
| Bilingual term parentheticals (EN + 中文) | ✅ | varies |

---

## Design principles

These are codified in the skills and govern every Q&A round:

1. **Interrogate vague answers.** "better / smoother / more professional" → the skill *must* follow up for a measurable standard. Accepting vague answers is a failure.
2. **Smart skipping.** Scan everything already collected; skip known elements.
3. **1–3 questions per round.** Long lists make users abandon.
4. **Cite examples to seed thinking** — but tell users not to copy verbatim.
5. **Subjective tasks require a rubric.** Without one, the reviewer agent has nothing to judge against.
6. **Context window health above all** (Phase 0a gatekeeper).
7. **Not every task is `/goal`-shaped** (Phase 0c) — and **not every task is worth a workflow** (`workflow-shaper`'s WORTH-IT gate). Both refuse rather than ship a doomed run.
8. **KICK-OFF is a self-cue.** After printing the kick-off block, the skill makes a tool call *in the same turn* — text alone doesn't reliably start the agent loop.
9. **Bilingual terminology.** First mention of any term writes `English (中文)`.

---

## The Handoff pattern

A novel mechanism — see [docs/handoff-pattern.md](docs/handoff-pattern.md). When a session is already context-strained, finishing the Q&A then running `/goal` compounds the problem. The skill packages in-progress state into `handoff.md` and tells the user to `/clear` + `/goal --resume <draft-name>` so the actual run lands on a full-context window. Generalizes to any long-running skill.

---

## Ecosystem

This repo is the **shaping layer** of a **five-public-repo AI-dev toolchain** by [dragon375014](https://github.com/dragon375014). Full topology, routing rules, and canonical skill ownership → [**ECOSYSTEM.md**](https://github.com/dragon375014/specmit/blob/main/ECOSYSTEM.md)

**One-command install** (drops all five tools into the right place):
```bash
npx specmit init
```

Siblings: [spec-sonar](https://github.com/dragon375014/spec-sonar) · [specmit](https://github.com/dragon375014/specmit) · [claude-skills-governance-meta](https://github.com/dragon375014/claude-skills-governance-meta) · [agent-work-board](https://github.com/dragon375014/agent-work-board) — each stands alone; install only what you need.

---

<a name="companion-repos"></a>
## Companion repos (姐妹 repo)

### spec-sonar — the spec side (規格側)

This repo is the **execution side** (how to run one task deep, or many units wide). [`spec-sonar`](https://github.com/dragon375014/spec-sonar) is the **spec side** (what to build): `idea-to-spec` converges a vague product idea into a spec via dark-zone Q&A, and `goal-decomposer` compiles that spec into a **dependency-ordered goal graph** — whose `G*.md` files use the **same five-element format** this repo's `/goal` produces, so they are directly executable via `/goal`-style KICK-OFF.

| You want | Use | Repo |
|---|---|---|
| Converge a full product idea into a spec | `idea-to-spec` | spec-sonar |
| Split a finished spec into dependency-ordered units | `goal-decomposer` | spec-sonar |
| Polish ONE task into an iterable goal prompt | `/goal` | this repo |
| Run the same check across N independent units | `workflow-shaper` | this repo |

Full chain: `idea-to-spec → goal-decomposer → goals/G*.md → execute each via /goal-style KICK-OFF (depth), fan out homogeneous sub-tasks via workflow-shaper (breadth)`.

### claude-skills-governance-meta — structure governance (結構治理)

This repo shapes **what to do** (the task spec). Its governance companion shapes **how it's built** (the structure):

> ### [`claude-skills-governance-meta`](https://github.com/dragon375014/claude-skills-governance-meta)
> Defensive + offensive governance patterns: an architecture-completeness gate (catches "I want to build X" before it's built), cross-layer **trace-lock** (lock a write→display chain so "changed A, forgot B" can't happen), and runnable defensive linters.

**They combine cleanly:**

- `workflow-shaper` here can be **dispatched by** that repo's `architecture-completeness-guardian` (the L0 gate decides "this is a many-unit job" → routes to the breadth shaper).
- A `/goal` or workflow you design here is **executed under** that repo's trace-lock + governance guard — so a multi-unit change stays SSOT-locked instead of drifting.
- Mental model: **this repo = the brief; that repo = the guardrails.** Use one for sharper task specs; add the other when your project is big enough to need structural governance (it ships a fitness check that tells you when).

If you only installed one, that's fine — each stands alone. But for a solo developer running long AI sessions on a real codebase, the pair covers both "shape the work" and "don't let the work corrupt the structure."

---

## Source material

The methodology draws from Anthropic's research on **context anxiety** and the implementer/reviewer architecture, Anthropic's **frontend-design** skill case study (the museum experiment → the 6-step rubric SOP), Karpathy on **evaluation > prompt engineering**, and the community insight that `/goal` succeeds or fails entirely on spec precision. The breadth half draws from Claude Code's **Dynamic Workflows** (background multi-agent orchestration). A transcript is in [examples/source-transcript.md](examples/source-transcript.md).

---

## Contributing

PRs and issues welcome. A few principles:

- **Don't add elements to the five-element framework.** It's stable on purpose. Open an issue first.
- **Don't widen `workflow-shaper`'s WORTH-IT gate to auto-approve.** The point is that it refuses cheap tasks.
- **Add example outputs to `examples/`.**
- **Translations welcome** (currently EN/繁中).
- **Test before PR** — run the skill end-to-end on your machine.

---

## License

[MIT](LICENSE). Do whatever you want with it; attribution appreciated but not required.

---

## Related

- [Claude Code skills documentation](https://code.claude.com/docs/en/skills)
- [Claude Code Dynamic Workflows](https://code.claude.com/docs/en/workflows)
- [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)
- [prompt-architect](https://github.com/ckelsoe/prompt-architect) — general prompt structurer

---

If either skill saved you a debugging session or a wasted `/goal` run, a ko-fi is appreciated but not expected. Everything here is MIT and free to use unconditionally.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/Z8C420A0VI)
