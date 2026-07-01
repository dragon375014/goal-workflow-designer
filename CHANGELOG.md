# Changelog

All notable changes to `goal-workflow-designer` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.0] - 2026-07-01

Closes the "goal guesses at an existing file's internals" blind spot, and lands a batch of field-applied improvements that had drifted into the running local copy without ever reaching this repo. Origin story: a 2026-07-01 crystallize-scan goal invented a non-existent `sessionData` field on `harvest-scan.mjs` — the whole SBE table built on air — caught only when a stronger model read the actual source. Under 0.3.0, Phase 0b would have forced the Explore scan to return `harvest-scan.mjs`'s real `footprint / insight / content` shape, blocking the guess at design time (a real counter-example, not a hypothetical).

### Added

- **`goal`: Phase 0b mutation-aware read-source branch (Step 3)** — when a goal's outcome is "modify / extend an existing file or module", the codebase scan flips from *optional* to **REQUIRED**, and the Explore subagent must return the target file's **real data contract** (資料結構 / 函式簽名 / I/O shape / external-interface-vs-internal-implementation gap), which is written into goal.md's `Codebase Context`. Greenfield / pure copy-editing goals keep the scan optional (no over-scanning). This is the precise fix for goals that infer a file's internals from its output / log instead of reading the source.
- **`goal`: Honesty rule "Don't fabricate the codebase either"** — sister clause to "Don't fabricate answers for the user". Any data field / function signature / interface cited in goal.md's `Codebase Context` or SBE table must be **read from source in Phase 0b**, never inferred; a guessed shape = skill failure (symmetric with the SBE 「沒真實數字 = skill failure」rule), with an explicit stop-signal for when you're citing something you haven't opened.
- **`goal`: verification SBE 例證化 discipline (Step 5.2 + Honesty Rules)** — branch / numeric / eligibility / state tasks must emit an SBE table (≥3 rows of 情境 × 具體輸入 → 預期輸出, real numbers, ≥1 happy + 1 boundary + 1 fallback); an abstract "calculation correct / tests pass" is a skill failure because the reviewer agent + downstream specmit scorecard get no re-runnable criterion. (Shipped in commit `cd70b3c`, previously un-changelogged — folded in here.)
- **`goal`: Step 5.7 — Workflow & relation reality-check** — for any UI / operator-workflow task, four probes beyond 5.6's blocked-state coverage: entry-point enumeration, current-workflow walkthrough, relation direction, existing / migration data. The single most common feature miss is shipping to one surface while the operator's real entry is another. (Field-backported from the running local copy.)
- **`goal`: two Honesty rules** — decisions go through `AskUserQuestion`, not prose (collapsing several forks into a text list + 「全照建議」silently strips the forcing function); and run Step 5.7 for any UI / operator-workflow task. (Field-backported.)

### Changed

- **`goal`: core principle #5 expanded — "1–3 questions per round — but loop, never collapse"** — never dump several decisions into a prose list + 「全照建議 / all recommended」to save clicks; each genuine fork gets its own `AskUserQuestion` round with the recommended choice first. (Field-backported.)

## [0.2.1] - 2026-06-10

Ecosystem conflict fixes — driven by a real-world conflict analysis run by the companion repo [spec-sonar](https://github.com/dragon375014/spec-sonar)'s Conflict Analysis Mode, which scanned the 4 skills (goal, workflow-shaper, idea-to-spec, goal-decomposer) coexisting in one environment and found 5 conflicts. Full report: spec-sonar `examples/conflict-report-goal-workflow-designer.md`.

### Added

- **`goal`: outbound routing section ("When NOT this skill")** — the systemic finding was that `goal` had zero outbound deferrals while owning the most generic name + the `/goal` command, making it the ecosystem's accretion point. It now conditionally defers: a full software product → `idea-to-spec` (if installed); a finished spec to decompose into a dependency graph → `goal-decomposer` (if installed). Deferrals are install-conditional so standalone installs keep working. (Fixes conflicts C1 + C3.)
- **`goal`: blocked-state / surface-coverage question (Step 5.6)** — backported from field use: UI / action-gating tasks are now asked about blocked states and rendering-surface coverage, the single most common UI miss.
- **`workflow-shaper`: symmetric SKIP clause** — heterogeneous modules with a dependency order (A before B) → `goal-decomposer` (if installed); a flat fan-out would lose the ordering. The one-second test now has three arms: one thing → `/goal`; same check across many → workflow; many different things with dependency order → `goal-decomposer`. (Fixes conflict C2.)
- **README (EN + tw): spec-sonar companion section** — documents the spec-side / execution-side split and the shared five-element goal format that makes spec-sonar's `G*.md` files directly executable via `/goal`-style KICK-OFF.

## [0.2.0] - 2026-05-29

Turns `skill-to-goal` into `goal-workflow-designer` — a two-skill repo covering both task-shaping axes (depth + breadth).

### Added

- **`workflow-shaper` skill** (breadth) — shapes an already-decided task into a runnable Claude Code Dynamic Workflow. Front WORTH-IT gate (5 criteria → green/yellow/red), fan-out shape recommendation (parallel / pipeline / map-reduce), paste-ready `workflow ...` prompt + draft meta block. Maps `/goal`'s five elements onto workflow stages. Refuses to emit a workflow prompt on a red light.
- **KICK-OFF self-cue (backported into the `goal` skill)** — after printing the kick-off block, the skill now makes a tool call *in the same turn* to start round 1, instead of ending the turn and waiting for the user to type "continue". New core principle #9; wired into Step 7 (`--use` run) and Step 11 (final run).
- **AI-executable install** ([INSTALL.md](INSTALL.md)) — paste the repo link into a session and the agent installs both skills to `~/.claude/skills/` and wires the triggers. Mirrors the companion repo's onboarding pattern.
- **Companion-repo cross-link** — README documents how this pairs with [claude-skills-governance-meta](https://github.com/dragon375014/claude-skills-governance-meta) (this = the brief; that = the guardrails).
- **Ko-fi link.**

### Changed

- **Repo renamed `skill-to-goal` → `goal-workflow-designer`** (GitHub keeps a redirect from the old name).
- **Restructured to a `skills/` layout**: the goal skill moved from the repo-root `SKILL.md` to `skills/goal/SKILL.md`; `workflow-shaper` lives at `skills/workflow-shaper/SKILL.md`. Install now copies per-skill folders instead of cloning the whole repo into `~/.claude/skills/goal`.
- README reframed around the depth/breadth pair, with an explicit "designer, not the engine, not required to use /goal" clarifier.

## [0.1.0] - 2026-05-25

Initial release.

### Added

- **Five-element framework Q&A** (Phase B) — guided collection of outcome, verification, constraint, iteration policy, and error handling for any task
- **Rubric 6-step SOP** (Phase C) — for subjective tasks: baseline check → frown points → dimensions → "Never do X" rules → diversity directions → output rubric.md
- **Phase 0a — Context Window Health Check** — gatekeeper at skill start; if user's session is context-strained, advise `/clear` or pack a handoff before continuing
- **Phase 0b — Codebase Scan** — optional Explore-subagent scan of the working directory, isolated from the main conversation context; scan summary is woven into later Q&A
- **Phase 0c — Goal-Eligibility Check (3-light system)** — evaluates four risk dimensions (external credentials, information completeness, human-only decisions, external blocking) before sinking time into Q&A. Yellow → list prerequisites and offer handoff; Red → stop and suggest alternatives
- **Handoff mechanism** — `/goal --resume <draft-name>` lets a context-strained Q&A pause and resume cleanly in a fresh session; see [docs/handoff-pattern.md](docs/handoff-pattern.md)
- **Smart skipping** — every round scans collected answers and skips already-known elements
- **Vague-answer interrogation** — "better / smoother / more professional" must be followed up for a measurable standard; codified as a core skill rule
- **Objective vs subjective task branching** — different question paths based on task nature
- **Bilingual terminology** (EN + 繁中) — first mention of any technical term writes `English (中文)`; skill detects user language and conducts Q&A in matching language
- **Three trigger paths** — `/goal <description>`, `/goal --use <name>`, `/goal --resume <draft-name>`, plus natural-language triggers
- **MIT License**

### Known limitations

- No-argument `/goal` is intentionally unsupported (forces users to give at least one sentence)
- Skill cannot spawn a new interactive Claude Code session — handoff requires the user to manually run `/clear` + `/goal --resume`
- Phase 0b codebase scan uses keyword extraction from the initial description; complex multi-domain tasks may produce noisy scans
- Currently bilingual EN/繁中 only — other-language support requires community translation PRs
