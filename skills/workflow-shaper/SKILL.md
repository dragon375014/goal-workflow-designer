---
name: workflow-shaper
description: Shapes an already-decided task into a runnable Claude Code Dynamic Workflow (a background JavaScript orchestration script that fans out tens-to-hundreds of fresh-context subagents, 16 concurrent / 1000 cap, intermediate results kept out of the main context, only the final synthesis returns). This skill does NOT run the workflow — it does three things: (1) a front WORTH-IT gate that decides whether the task is even worth a workflow, (2) recommends a fan-out shape (parallel / pipeline / map-reduce), (3) emits a paste-ready `workflow ...` prompt + a draft meta block. The breadth (廣度) counterpart to /goal's depth (深度). 把「已決定要做」的任務塑形成可跑的 workflow，先過值不值得開的閘門，值得才產出可貼上的 workflow prompt。
trigger: workflow-shaper
---

# workflow-shaper — Shape a task into a runnable Dynamic Workflow (把任務塑形成可跑的 Dynamic Workflow)

Turns "I've decided to do X" into a workflow that is **worth running and well-organized**. Runs a WORTH-IT gate first; only if it passes does it emit a paste-ready `workflow ...` prompt.

把「我已經決定要做某件事」變成一支**值得跑、組織得當**的 workflow。先過 WORTH-IT 閘門（值不值得開），值得才產出可貼上的 `workflow ...` prompt。

> **This skill shapes; it does not execute.** Its output is handed back to you / the orchestrator to paste. It never calls the Workflow tool itself.
> 本 skill 只塑形、不執行。產出交回給你 / 主 orchestrator 貼上，不自己 call Workflow tool。

## Relationship to /goal — depth vs breadth (與 /goal 的關係 — 深度 vs 廣度)

These two skills are siblings. They both take a vague task and shape it into a precise, executable spec — but along **different axes**:

| | `/goal` | `workflow-shaper` (this) |
|---|---|---|
| Axis | **Depth (深度)** — one worker + a rubric, iterate to a quality bar | **Breadth (廣度)** — a foreman's org-chart, many workers in parallel, cross-checked |
| Solves | "one thing done right, deep, and converged" | "a pile of things split, parallelized, cross-verified, beyond one context window" |
| Gate | `/goal` Phase 0c Goal-Eligibility (is the task `/goal`-shaped?) | this skill's WORTH-IT gate (is the task worth a workflow?) |

**They compose**: a workflow is the for-loop, a goal is the loop body. A workflow's verify stage can embed a goal-style rubric. The one-second test for which to reach for: **count how many units** — one thing to perfect → `/goal`; the same check across many things → a workflow; many *different* things with a dependency order (A must finish before B starts) → `goal-decomposer` (if installed), not a flat fan-out.

---

## Core principles (核心原則 — read before invoking)

1. **Never force a workflow prompt on a RED light.** That would nudge the user to burn dozens-to-hundreds of subagents on something an inline run handles. Honesty over delivery.
2. **A YELLOW light always offers a middle path** (split into several short workflows / inline-run one unit first to validate), never a forced binary.
3. **Bilingual terminology / 術語雙語化**: first mention of any technical term writes `English (中文)`, e.g. `fan-out (扇出並行)`, `reducer (彙整代理)`, `barrier (同步點)`.
4. **Shape, don't run.** Output is handed back; this skill does not invoke the Workflow tool.

---

## When this fires (TRIGGER) / When it doesn't (SKIP)

**TRIGGER when** (semantic match, not literal):
- The user says a workflow keyword: "use a workflow", "fan out / parallel-scan", "run a workflow over all the X", "save it as a `/<name>` workflow".
- The user describes **the same check / same change applied to many independent units**: "audit all 20 API endpoints for missing auth", "check every migration for the missing GRANT", "verify all N trace tests", "extract all 30 modals into a shared component".
- A high-stakes architecture decision needs **adversarial second opinions / multi-angle cross-checking**: "stress-test this schema change from a few independent angles".

**SKIP when**:
- Single-unit work (1 file / 1 bug / 1 endpoint) → a plain skill or `/goal` is enough; a workflow wastes tokens.
- You need intermediate results to stay in the main context for follow-up, or you need mid-run human sign-off → a workflow discards intermediate state and runs in the background, which gets in the way. Use inline `/goal`.
- Pure copy / pure styling / a one-line typo.
- The user is still at the **"should we even do this?" decision layer** → that is a requirements / architecture-gate question (if you adopted the companion `claude-skills-governance-meta`, that's its `architecture-completeness-guardian`'s job), not this skill. This skill assumes the decision is made; it only answers "breadth-or-not, and how to organize."
- The units are **heterogeneous modules with a dependency order** (A must finish before B can start — e.g. a spec decomposed into modules, not N same-shaped checks) → that's `goal-decomposer`'s job (if installed): it compiles a spec into a dependency-ordered goal graph. A workflow is a flat fan-out and would lose the ordering.

---

## Step 1 — Threshold check (門檻檢查)

Confirm two things; bail early on either:
- **Decided?** If the user is still on "whether to do it / which version", that's the decision layer — not this skill. Stop.
- **Non-trivial?** Pure copy / styling / one-line typo → just do it, no workflow. Stop.

Pass → Step 2.

## Step 2 — WORTH-IT gate (核心：5 criteria → 3-light)

Judge each; give openWorkflow / stayInline + a one-line reason:

| # | Dimension | openWorkflow if… | stayInline if… |
|---|---|---|---|
| 1 | **Unit count (單位數量)** | ≥5 same-shaped, independent, parallelizable units (no ordering dependency) | 1 unit, or N<5 and doing them one by one is painless |
| 2 | **Cross-checked quality (交叉驗證)** | high-stakes / irreversible decision needs a fresh-context verifier to adversarially attack it, or results must be cross-compared to be trusted | low-stakes, reversible, single judgment is enough; or verification is just running a test/lint (faster inline) |
| 3 | **Token cost (成本權衡)** | the context saved (N intermediate results NOT flooding the main window) + wall-clock compressed by parallelism clearly outweighs the extra tokens | task is light / one-off, or you're context- or quota-constrained right now |
| 4 | **Intermediate results (中間結果歸屬)** | you only want the final synthesis / a PASS-FAIL list; each unit's reasoning can be discarded (workflows keep it OUT of context by design) | you need each unit's intermediate output in the conversation to follow up on or hand-pick |
| 5 | **Mid-run sign-off (中途簽核)** | fully automatable, machine-judgeable success bar (schema / test / rubric score), no human nod needed mid-run | there's a "continue? / which path? / I must eyeball this" human decision point mid-run |

**Verdict**:
- 🟢 **Green** (key criteria all green, esp. #1 #4 #5) → proceed to Steps 3-4.
- 🟡 **Yellow** (borderline, e.g. N=4, or one mid-run sign-off point) → offer a middle path: (a) split into several short workflows with human sign-off between them; (b) inline-`/goal` one unit first to validate, then decide on fan-out. **Do not force a binary.**
- 🔴 **Red** (single unit / needs intermediate context / needs human loop / quota-sensitive) → **explicitly bail, recommend a plain skill or `/goal`, emit NO workflow prompt.**

## Step 3 — Recommend a fan-out shape (green light only)

Pick one based on task structure; include an ASCII shape diagram marking the verify stage and loop boundary:

- **parallel (純並行, has a barrier)**: N independent units, no ordering → fan out all N, each returns its result, all must return before moving on. Use when a downstream stage needs the entire result set at once (dedup / merge / cross-compare).
- **pipeline (流水線, no barrier)**: ordered dependency (extract SSOT → compare → verify) → stages chained; item A can be in stage 3 while item B is still in stage 1. **The default for multi-stage work.**
- **map-reduce (most common)**: fan out each unit to produce a structured result (map) → a reducer agent cross-checks + synthesizes into a final shape (reduce) → optionally loop-until-dry on the FAIL subset.

## Step 4 — Emit a paste-ready `workflow ...` prompt + draft meta block

Produce three parts:
1. **Trigger line**: open with `workflow: <one-line task>` so the orchestrator recognizes workflow mode.
2. **Five-element-mapped content** (see table below): final return shape (JSON schema) / verify-stage prompt (with embedded rubric) / concurrency + budget caps / loop-until-dry condition / null-drop + stall-timeout error handling. Each fan-out subagent gets a unit-prompt template (with its constraint / no-go list, read-only declaration where appropriate).
3. **Draft meta block + save suggestion**: ask whether to save it as `.claude/workflows/<name>` (→ becomes a `/<name>` slash command, version-controlled "governance-as-code"); include a `name` + `description` frontmatter draft.

### Five-element → workflow-stage mapping (maps /goal's framework onto breadth)

| `/goal` element | → workflow equivalent |
|---|---|
| **Outcome** (what "done" looks like) | the reducer's **final return shape** (structured synthesis object). Each unit's intermediate reasoning is deliberately kept out of this shape. |
| **Verification** (how to prove it's done) | a standalone **adversarial-verify stage**: a fresh-context verifier that never saw the implementation attacks each unit's output with a schema + rubric, then cross-checks for contradictions. `/goal`'s single-worker rubric becomes the verify stage's embedded scorecard. |
| **Constraint** (what must not be touched) | **concurrency / budget caps + no-host-access + per-subagent no-go list**. `/goal`'s "don't touch X" is written into each fan-out subagent's prompt; plus the workflow-level hard caps (16 concurrent / 1000 total / read-only). |
| **Iteration policy** (what each round does) | **loop-until-dry / budget loop**: the reducer feeds the previous round's FAIL list as the next round's fan-out input, converging until clean or budget is exhausted. |
| **Error handling** (when to pause and report) | **`.filter(Boolean)` + null-drop + stall timeout**: a failed subagent returns null, the reducer drops it, and the final shape reports them in an `errored: [...]` section — rather than the whole batch halting mid-run. |

---

## Worked example (green light → full flow)

Scenario: the user says "I changed the shared payment-formatting helper; confirm none of my N order/checkout views drifted off it and started re-implementing the rendering." Matches TRIGGER (same check across many independent units).

**Step 2 WORTH-IT**: N same-shaped independent views (#1 green); money display is medium-risk and "one view silently drops the discount line" is a recurring bug class → needs a verifier (#2 green); reading N files serially would flood the main context, fan-out keeps the reads in subagents and returns only conclusions (#3 green); you only want the gap list, per-view reads are disposable (#4 green); pure scan, machine-judgeable success bar (#5 green) → **🟢 all green, open workflow.**

**Step 3 shape** = map-reduce:

```
                 [ orchestrator ]
                        │ fan-out N (parallel, cap 16)
   ┌──────┬──────┬─────┴────┬──────────┐
 view 1  view 2  view 3   view 4  ...  view N   ← map: each reads its view + compares to the shared SSOT helper
   └──────┴──────┴──────────┴──────────┘
                        │ each returns { unit, status, violations[] } (failure → null)
                 [ verifier agent ]   ← adversarial verify: independently re-checks each violation against the SSOT
                        │
                 [ reducer ]  → final shape: gap list
                        │ loop-until-dry: any FAIL → fan out the FAIL subset once more
                 returns to main context (intermediate reads discarded)
```

**Step 4** → emit the `workflow: audit N views against the shared SSOT helper` prompt with the final return shape, the verifier-stage prompt, the per-view read-only unit template, the loop-until-dry condition, and a draft meta block suggesting `.claude/workflows/<name>` so it becomes a re-runnable `/<name>` command.

---

## Honesty Rules (誠實守則)

- **A WORTH-IT RED light never emits a workflow prompt** — recommend a plain skill or `/goal` instead. Better to not deliver than to nudge the user into burning hundreds of subagents on something an inline run handles.
- **A YELLOW light always offers a middle path**, never a forced binary.
- **Don't fabricate fan-out units.** If the unit list isn't clear, ask the user to confirm it, or spawn one exploratory agent to enumerate the units first — don't make them up.
- **State the workflow's inherent limits honestly**: no mid-run human sign-off (it runs in the background), no follow-up on intermediate results, no resume across sessions, usage counts against plan / rate limits.
- **Shape, don't run** — output is handed back; this skill never calls the Workflow tool itself.
