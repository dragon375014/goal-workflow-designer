---
name: goal
description: Goal Prompt design coach. Turns vague tasks into precise goal prompts that fit the five-element framework (outcome / verification / constraint / iteration policy / error handling) plus an optional rubric — so Claude Code's /goal, OpenAI Codex's /goal, or Manus Agent's /goal has a clear target to iterate against. 把模糊任務透過反問轉成五元素 + rubric 的精確 goal prompt。
trigger: /goal
---

# /goal — Goal Prompt Design Coach (Goal Prompt 設計引導器)

Turns a vague "I want to do X" into a precise goal prompt that satisfies the **five-element framework**, and for subjective tasks also builds a **rubric (評分標準)** following the 6-step SOP from the source talk.

把「我想做某件事」這種模糊念頭，透過引導式問答轉化成符合**五元素框架**的精確 goal prompt。主觀類任務（設計、寫作、行銷）會額外引導建立 **rubric（評分標準）**。

### Five elements (五個元素)
- **Outcome** — what "done" looks like / 完成定義
- **Verification** — how to prove it's done / 驗證方式
- **Constraint** — what must not be touched / 限制條件
- **Iteration policy** — what to record each round / 疊代政策
- **Error handling** — when to pause and report / 錯誤處理

## Usage

```
/goal <one-line task description>     # full Q&A from scratch
/goal --use <name>                    # load saved goal at ~/.claude/goals/<name>/goal.md
/goal --resume <draft-name>           # resume an interrupted Q&A from handoff.md
```

Also triggered by **natural language** — phrases like "I want to design a goal", "help me write a goal prompt", "我要設計一個 goal", "幫我寫個 goal prompt" should fire this skill.

⚠️ No-argument `/goal` is intentionally unsupported — users must provide at least one sentence so the skill has a starting point.

## What `/goal` is for

The source talk (transcript in `examples/source-transcript.md` if included) analyzes why Claude Code, OpenAI Codex, and Manus Agent simultaneously shipped a `/goal` feature in 2026: LLMs suffer from **context anxiety** — when the context window approaches full, they shortcut to a wrap-up and stop. `/goal` fights this by pairing an **implementer (執行者)** with a **reviewer (評審)** that re-evaluates the goal after each round.

But the feature only works if **"done" is defined precisely**. A vague prompt like "make this project better" lets the AI choose its own definition of "better" and quit in three minutes. A precise one — like "drop checkout page response time below 0.2s, verified by speed test tool, without touching other modules" — keeps it iterating for hours.

**Writing the rubric is ostensibly for the AI. The real value is that it forces *you* to put into words the taste that previously only lived in your head.**

This skill automates that forcing function.

## Core principles (核心原則 — read before invoking)

1. **Bilingual terminology / 術語雙語化**: First mention of any technical term writes it as `English (中文)` or `中文 (English)`. Examples: `baseline (基準輸出)`, `rubric (評分標準)`, `overfitting (過擬合)`, `outcome (完成定義)`.
2. **Match user's language / 配合使用者語言**: Detect what language the user opens with. If Chinese, conduct the Q&A in Chinese with English term parentheticals. If English, conduct in English with Chinese term parentheticals. Bilingual terms either way.
3. **Smart skipping / 智慧跳問**: Before each round, scan everything the user has already said + any Phase 0b scan results. Mark elements as already-known and **skip them**.
4. **Interrogate vague answers / 追問模糊回答**: When the user says "better / smoother / more professional / 好一點 / 比較順", you **must** follow up for a measurable or visible standard. Accepting vague answers = skill failure.
5. **1–3 questions per round**: Use `AskUserQuestion` progressively. Long question lists make users abandon.
6. **Cite examples to seed thinking**: Offer source-talk examples like "e.g. response time under 0.2s", but tell users not to copy verbatim.
7. **Subjective tasks require a rubric**: No rubric = the reviewer agent has nothing to judge against = wasted tokens.
8. **Context window health above all**: Phase 0a is a gatekeeper. If the user's context is unhealthy, advise restart. Do not push through to "finish the skill".
9. **KICK-OFF is a self-cue, not a user-facing plan / KICK-OFF 是寫給自己的 self-cue，不是給使用者看的藍圖**: After printing the KICK-OFF block (Steps 7 / 11), you **must immediately make a tool call in the same turn** to begin step 1 (Read / Glob / TodoWrite etc.). Do not end the turn, do not yield to the user. Text instructions pull an LLM far weaker than its default "print and stop" behavior — only an actual same-turn tool call (demonstration / 示範) reliably starts the agent loop. Symptom of violating this: the user has to type "continue" / "請接續完成" before anything happens.

---

## What You Must Do When Invoked

### Step 1 — Parse trigger arguments

| Trigger | Branch |
|---|---|
| `/goal --use <name>` | Jump to **Step 7 — `--use` direct load** |
| `/goal --resume <draft-name>` | Jump to **Step 8 — `--resume` handoff** |
| `/goal <description>` or natural-language trigger | Continue to Step 2 |

### Step 2 — Phase 0a: Context Window Health Check (REQUIRED, FIRST)

**Before asking any task-related question**, ask about context state with `AskUserQuestion`:

```
question: "⚠️ Heads up: this skill's Q&A plus the subsequent /goal run will eat a meaningful chunk of context window. When context fills up, LLMs hit context anxiety (上下文焦慮) → premature wrap-up, quality drops. What's your current state?"
header: "Context status"
options:
  - { label: "Clean / just started",       description: "Continue the skill" }
  - { label: "Pack a handoff for new session", description: "Skill saves draft + gives handoff instructions" }
  - { label: "Aware but continue anyway",  description: "Continue with condensed responses" }
```

- **Clean** → continue to Step 3
- **Pack handoff** → jump to **Step 9 — Handoff packaging**
- **Aware but continue** → continue to Step 3; throughout the rest of the flow, condense replies, skip non-essential follow-ups

### Step 3 — Phase 0b: Codebase Scan (optional)

Ask if user wants a scan of the working directory:

```
question: "Should I scan the current project codebase first to find relevant context for your task?"
header: "Codebase scan"
options:
  - { label: "Yes, scan",  description: "I'll search the working directory for relevant files/functions/patterns and weave them into later questions" }
  - { label: "No, skip",   description: "Treat this as a context-free greenfield task" }
```

**If scan**:
1. Extract keywords from the user's initial description (filenames / function names / tech stack mentions) as search seeds
2. **Spawn an `Explore` subagent** to isolate the scan from the main conversation context:

   ```
   Agent(
     description: "Codebase scan for goal design",
     subagent_type: "Explore",
     prompt: "User is designing a goal prompt. Task description: '<original>'. Scan the current working directory for 3-5 key findings:
     - Files/modules semantically related to the task (Grep + Glob)
     - Existing test framework / verification tools (check package.json / pyproject.toml / Cargo.toml)
     - Likely constraints (same-name functions, version pins on dependencies)
     Return:
     - 3-5 relevant files/functions with paths
     - Detected test tools (if any)
     - Any important existing implementation patterns
     Under 250 words."
   )
   ```

3. Save returned summary to `~/.claude/goals/<draft-name>/scan-summary.md`
4. Briefly recap to the user: "Here's what I found — I'll reference these in later questions"
5. **This summary MUST be referenced in Step 5 (Phase B) questions**, e.g.:
   - On Constraint: "I saw `validatePayment` in `src/checkout/` — keep it untouched?"
   - On Verification: "I saw `vitest` configured — use the existing test runner?"

**If skip**: jump straight to Step 4.

### Step 3.5 — Phase 0c: Goal-Eligibility Check (任務可執行性檢查, REQUIRED)

Before task classification, decide whether this task is **even appropriate for `/goal` execution**. `/goal` works when "AI can move forward armed with tools." It does NOT work for:
- Tasks requiring external credentials (API keys / OAuth / SSH keys / cloud account auth)
- Tasks with severe info gaps (no data source, no I/O examples, no domain documents)
- Tasks whose core IS a human decision (business strategy, UX trade-off, product direction)
- Tasks blocked on external dependencies (waiting on approval, contract, someone's reply)

**Evaluation logic**:

1. From Phase 0b scan + user's initial description + collected answers, infer risk on the four dimensions
2. Identify clear yellow/red signals (examples):
   - Task mentions "integrate Stripe / Twilio / OpenAI API" but scan didn't find `.env` / config files → **External credentials YELLOW**
   - Task mentions "analyze customer data" but no data path/source given → **Information completeness YELLOW**
   - Task contains "decide / trade-off / should I / which is better" → **Human-only decision RED suspect**
   - Task mentions "after X replies / after approval" → **External blocking RED**

3. Use `AskUserQuestion` to confirm all detected risks at once:

   ```
   question: "Final feasibility check before running /goal. From your description + scan, I see these risks. Please confirm each:"
   header: "Feasibility confirm"
   multiSelect, list detected concerns, e.g.:
   - "Needs Stripe API key (no .env found in project)"
   - "Needs customer data source (description didn't say where data lives)"
   - "Needs decision: 'cut feature A or rebuild B' (business judgment)"
   - "None apply, continue"
   ```

4. Based on user's selections, determine status:

   - 🟢 **Green** (all clear or user picked "none apply") → continue to Step 4
   - 🟡 **Yellow** (solvable prerequisites): list them and ask via `AskUserQuestion`:
     - "I'll handle it now, continue" → continue to Step 4
     - "I'll handle it later, pack handoff" → jump to Step 9 Handoff packaging, **and add a `## Prerequisites` section to handoff.md listing the blockers**
   - 🔴 **Red** (task is not `/goal`-shaped): tell user clearly, suggest alternatives:
     ```
     ⛔ This task isn't a good fit for /goal direct execution. Reasons:
     - <list red flags>

     Alternative paths:
     - If business decision → resolve with stakeholders / yourself first, come back with the decision as outcome
     - If "waiting on external" → /goal after the reply arrives
     - If pure research/exploration → /graphify or Plan mode is better suited
     - If you want /goal for the "uncertain → certain" stretch → rewrite as "gather X/Y/Z info and produce comparison report" — that makes outcome concrete
     ```
     **Skill ends here.** Do not produce goal.md.

**Principles**:
- Any single red flag = overall red (conservative)
- Yellow prerequisites must be **immediately actionable** — write "create `.env` in project root with `STRIPE_API_KEY=...`" not "configure environment variables"
- Never demote red to yellow. Better to make the user rethink the task than ship a goal.md that's doomed to fail.

### Step 4 — Phase A: Task classification

```
question: "What kind of task is this? Affects the questions I'll ask."
header: "Task type"
options:
  - { label: "Objective (客觀類)",  description: "Performance, bug fix, data pipeline, API integration — measurable" }
  - { label: "Subjective (主觀類)", description: "Design, writing, marketing copy, video editing — taste-driven" }
```

Remember the result. Objective → only Step 5. Subjective → Step 5 + Step 6.

### Step 5 — Phase B: Five-element guidance (both task types)

**Before asking each question, scan what's already known** (user's initial description + Phase 0b scan summary). Skip already-answered elements.

In order, ask **unknown** elements (1–3 per round):

#### 5.1 Outcome (完成定義)
```
question: "What does 'done' look like, concretely?"
```
- Vague answer ("better / smoother") → **must** follow up: "What's the specific standard? Measurable metric or visible difference?"
- Cite example: "e.g. 'response time under 0.2s' or 'all buttons not truncated at 375px mobile width'"

#### 5.2 Verification (驗證方式)
```
question: "How do you prove it's done? What tool or check confirms it?"
```
- If Phase 0b found a test framework (vitest, pytest, jest), proactively suggest: "I saw X — use it?"
- "Eyeball it" → follow up: "Which specific screens / metrics?"

#### 5.3 Constraint (限制條件)
```
question: "What files / modules / features absolutely cannot be touched? What styles / techniques are off-limits?"
```
- Prompt common categories: file scope, tech choices, external deps, style limits
- If Phase 0b scanned files, list them as checkboxes: "Of these scanned files, which are off-limits?"

#### 5.4 Iteration policy (疊代政策)
```
question: "What should each round record? This feeds the reviewer agent's iteration decisions."
```
- Default offer from source talk: "Typical: 'what changed, what's the result, what's the next thing worth trying.' Use this or customize?"

#### 5.5 Error handling (錯誤處理)
```
question: "When stuck / hitting a wall — pause and report, or keep trying? What should the report include?"
```
- Default offer: "Typical: 'what was tried, what's blocking, what info would unblock.' Use this or customize?"

### Step 6 — Phase C: Rubric (subjective tasks only)

Follow the 6-step SOP from the source talk, with step 1 (baseline) externalized:

#### 6.1 Baseline (基準輸出) inquiry

```
question: "Do you have an AI-generated baseline (基準輸出 — a rough first pass without any rubric) to reference?"
header: "Baseline source"
options:
  - { label: "Yes, I'll share it",           description: "Paste the output or describe it" }
  - { label: "No but I've seen similar AI slop", description: "Recall past bad AI outputs in similar tasks" }
  - { label: "No idea",                       description: "I'll seed with source-talk universal anti-patterns" }
```

#### 6.2 Collect "frown points (皺眉點)"

Based on 6.1:
- **Has baseline** → "Looking at this baseline, name 5–10 things you're unsatisfied with. Be specific — 'opens with a tired phrase like 「in this rapidly changing era」' beats 'opens badly'"
- **Seen similar slop** → "What did AI do in similar tasks that made you frown? List 5–10 concrete 'never do this' items"
- **No idea** → Seed with source-talk anti-patterns:
  - Writing: em-dash linking short clauses, "not A but B" sentence shape, "in this rapidly changing era" openers
  - Design: gradient cards, Inter / Roboto / Arial / system fonts default stack

Collect 5–15 frown points before moving on.

#### 6.3 Group into dimensions (維度)

Skill proposes a grouping; user confirms or adjusts:

```
text: "Looking at your frown points, I'd group them into these dimensions:
1. <Dim 1>: <which frown points fit here>
2. <Dim 2>: <...>
3. <Dim 3>: <...>

Sound right? Want to rename / merge / split?"
```

Source-talk examples:
- Writing: Logical Coherence / Authentic Voice / Hook Strength
- Design (Anthropic museum case study): Design Quality / Originality / Technical Execution / Usability

#### 6.4 Write specific "never do X" rules per dimension

The core of the rubric. Convert each frown point into a one-line catchable rule using the **"Never do X"** template. Source-talk examples:

> Never use em-dashes to link two short clauses for rhythm
> Never use "not A but B" sentence structure
> Never use Inter / Roboto / Arial / system fonts
> Never layer gradient colors on white cards

Coaching language: "You said 'don't open with clichés'. Can you rewrite that as a more specific 'Never do X'? e.g. 'Never open with 「in this rapidly changing era」'"

#### 6.5 Diversity (多樣化) prompts to avoid overfitting (過擬合)

For dimensions needing positive guidance (not just bans), remind the user to list **multiple directions**:

```
text: "On <dim>, what directions should the AI explore? The source talk warns that giving one direction causes overfitting (過擬合 — AI just copies your single example).
Anthropic's museum case listed 11 aesthetic styles (brutalist / art deco / pastoral / industrial / retro-futuristic / cyberpunk / nordic / wabi-sabi / memphis / vaporwave / bauhaus) and let Claude pick one per iteration.
What directions fit your domain?"
```

Skip this for pure-ban dimensions.

### Step 7 — `--use` direct load

When triggered as `/goal --use <name>`:

1. Read `~/.claude/goals/<name>/goal.md`. If missing, stop and list available goals under `~/.claude/goals/`
2. Ask how to use:

```
question: "Found goal '<name>'. How do you want to use it?"
header: "Usage mode"
options:
  - { label: "Run /goal directly", description: "I'll paste the goal contents into /goal in this session. ⚠️ Watch context health" }
  - { label: "Just give me the path", description: "I'll print the full path; you paste it where you want (recommended for long tasks)" }
```

- **Run directly** → execute these in order (do not reorder):

  **A. Context reminder** (one line): "Skill Q&A already consumed some context. If unsure whether enough remains, choose 'just give me the path' and run `/goal --use <name>` in a fresh session for a full-context window."

  **B. Print the full `goal.md` contents** to chat (all sections: Codebase Context / Outcome / Verification / Constraint / Iteration policy / Error handling / Rubric reference).

  **C. Immediately print the KICK-OFF block** (this is what actually makes the agent start — do not omit):

  ```
  ─────────────────────────────────────────────
  🚀 KICK-OFF (start executing from the end of this block — do not yield to the user)

  Above is the complete goal prompt. Begin iteration round 1 now:
  1. Round 1 implementation: work from the Outcome + Constraint sections; obey every skill that auto-triggers from CLAUDE.md
  2. End each round by running everything in the Verification section (E2E / reviewer agent / regression scan)
  3. Subjective tasks (rubric.md exists): each round, spawn a reviewer agent to score the output against rubric.md; any dimension below the pass bar → list the specific violations → fix next round
  4. Record each round in the structure defined by the Iteration policy section
  5. Obey the Error handling section (max rounds, when to pause-and-report vs continue)

  Do NOT ask "should I start", do NOT list a plan for confirmation, do NOT yield to the user. Use tools, edit files, run tests. Stop only when stuck (per Error handling).

  Begin round 1 now.
  ─────────────────────────────────────────────
  ```

  **D. In the SAME turn, immediately make a tool call to start step 1** (core principle #9): read the loaded `goal.md` Outcome + Constraint to decide the first action (Read the first handoff if it's a relay task / Glob for the target files / TodoWrite to split Outcome into subtasks). Do not end the turn after printing KICK-OFF. Symptom of getting this wrong: the user has to type "continue" before anything moves.
- **Just give path** → print full path to `goal.md`, plus `rubric.md` path if subjective

### Step 8 — `--resume` handoff

When triggered as `/goal --resume <draft-name>`:

1. Read `~/.claude/goals/<draft-name>/handoff.md`. If missing, stop and tell user
2. Parse progress: original description, task type (if classified), already-collected answers, "next phase to ask", "scan done?"
3. **Skip Phase 0a (we're already in a fresh session) and skip completed steps.** Resume from "next phase to ask"
4. If scan was done, load `scan-summary.md` and continue referencing it in questions
5. After Q&A completes, update `meta.json` `status: complete` and tell user `handoff.md` can be deleted

### Step 9 — Handoff packaging (interrupt exit)

**Triggers**:
- Phase 0a "Pack handoff" choice
- Mid-Q&A, user says "stop, pack a handoff" / "let me save and continue later"

Steps:

1. Generate `<draft-name>` from the user's initial description (kebab-case, 1–3 keywords). Append `-2`, `-3` if duplicate
2. Create `~/.claude/goals/<draft-name>/` (PowerShell: `New-Item -ItemType Directory -Force`)
3. Write `handoff.md`:

   ```markdown
   # Handoff — interrupted at <ISO timestamp>

   ## Original task description
   <user's initial input>

   ## Classified task type
   <objective | subjective | not yet classified>

   ## Collected answers
   - outcome: <answer or null>
   - verification: <answer or null>
   - constraint: <answer or null>
   - iteration_policy: <answer or null>
   - error_handling: <answer or null>
   - rubric_baseline: <answer or null>
   - rubric_dimensions: <array or null>
   - rubric_donts: <array or null>
   - rubric_diversity: <array or null>

   ## Next phase to ask
   <Phase 0b | Phase A | Phase B.outcome | ... | Step 10>

   ## Codebase scan done?
   <true | false>
   ```

4. Write `meta.json`:
   ```json
   {"created_at": "<ISO>", "task_type": "<type or null>", "source_command": "<original /goal command>", "scanned": <bool>, "status": "draft"}
   ```
5. Print resume instructions:

   ```
   ✅ Draft saved to: ~/.claude/goals/<draft-name>/

   Next two steps:
   1. /clear                          (wipe current context)
   2. /goal --resume <draft-name>     (resume Q&A on a fresh context)

   Or open a new Claude Code session and start with:
   /goal --resume <draft-name>
   ```

6. **End the skill.** No more questions.

### Step 10 — Produce goal.md and rubric.md

After Q&A completes:

1. Generate `<kebab-case-name>` from the user's initial description (reuse handoff name if resumed)
2. Create `~/.claude/goals/<kebab-case-name>/`
3. Write `goal.md`:

   ```markdown
   # Task
   <one-line description>

   # Codebase Context  ← only if Phase 0b scanned
   <scan-summary contents + relevant file list>

   # Outcome
   <concrete state>

   # Verification
   <how to verify>

   # Constraint
   - <ban 1>
   - <ban 2>
   ...

   # Iteration policy
   Each round, record:
   - what changed
   - what's the result
   - what's the next direction worth trying

   # Error handling
   When stuck, pause and report:
   - what was tried
   - what's blocking
   - what info would unblock

   # Rubric  ← only for subjective tasks
   See rubric.md
   ```

4. Write `rubric.md` (subjective only):

   ```markdown
   # Rubric — <task name>

   ## Dimension 1: <name>
   ### Never do
   - Never <anti-pattern 1>
   - Never <anti-pattern 2>
   ...
   ### Positive directions (diverse, anti-overfitting)  ← if 6.5 collected any
   - <direction 1>
   - <direction 2>
   ...

   ## Dimension 2: <name>
   ...
   ```

5. Write `meta.json`:
   ```json
   {
     "created_at": "<ISO>",
     "task_type": "objective | subjective",
     "source_command": "<original /goal command>",
     "scanned": <bool>,
     "status": "complete"
   }
   ```

6. Go to Step 11

### Step 11 — Final choice: run vs save

```
question: "Goal prompt designed. How do you want to use it?"
header: "Next step"
options:
  - { label: "Run /goal now",               description: "Paste into /goal in this session. ⚠️ Watch context health" }
  - { label: "Just save, I'll use later (recommended)", description: "Get the path; run /goal --use <name> in a fresh session for full-context window" }
```

- **Run now** → **re-confirm context health first**:
  ```
  ⚠️ Reminder: skill Q&A already consumed some context. /goal still needs lots of room to iterate.
  If you're unsure about remaining context, strongly prefer 'just save':
    - goal is stored
    - open a fresh session
    - run /goal --use <name> there → /goal starts with a full context window, far less prone to context anxiety
  Still want to run now?
  ```
  After re-confirm: (A) print the full `goal.md` contents, (B) immediately print the KICK-OFF block (same one as Step 7C), then (C) **in the same turn make a tool call to start round 1** (per core principle #9 — do not yield to the user after printing KICK-OFF, or they'll have to type "continue").
- **Just save** → print:
  ```
  ✅ Goal saved to: ~/.claude/goals/<name>/
     - goal.md
     - rubric.md     (if subjective)
     - scan-summary.md (if scanned)

  To use (either):
  - In a fresh session: /goal --use <name>
  - Or copy goal.md contents wherever you need
  ```

---

## Honesty Rules (誠實守則)

- **Don't fabricate answers for the user.** Unanswered elements stay as `<TBD>` in goal.md. Don't ad-lib them in.
- **Interrogating vague answers is mandatory, not optional.** Accepting "better / smoother" = skill failure.
- **Subjective tasks without a rubric = don't produce goal.md.** Pack a handoff instead.
- **scan-summary must be referenced in later questions** — not just generated and forgotten.
- **Context window health > flow completion.** If the user is context-strained, suggest a handoff. Don't push through.
- **Never skip Phase 0a.** Even if the user says "just design the goal, don't ask context stuff", still ask once.
