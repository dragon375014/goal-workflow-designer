# The Handoff Pattern

> A reusable pattern for long-running Claude Code skills that meaningfully consume context window.

This document describes the **Handoff Pattern** that `goal-workflow-designer` introduces. The pattern is extracted from `SKILL.md` and documented separately so other skill authors can adopt it without needing to reverse-engineer the implementation.

---

## The problem

Anthropic's own research describes **context anxiety** — when an LLM's context window approaches full, it begins to "wrap up" prematurely. It will close a half-finished task with a summary, skip remaining steps, and quit. The closer to the context limit, the more aggressive this behavior becomes.

Long-running skills compound the problem. Consider the lifecycle:

```
User opens session ──────────────────────────────────────────────►
   │
   │  (some prior conversation consumes context already)
   │
   ├─► User invokes skill
   │     ┌─► Skill asks 5–15 questions (each round consumes context)
   │     ├─► Skill may run subagent (returns summary, more context)
   │     ├─► Skill writes output files (paths in context)
   │     └─► Skill ends
   │
   ├─► User triggers the actual task (e.g. /goal)
   │     └─► Task needs LOTS of context room to iterate
   │
   └─► At this point context might already be 60–80% full
         → /goal hits context anxiety early
         → Premature wrap-up
         → User gets a half-baked output and wonders why
```

The skill that was supposed to *improve* the user's outcome ends up *poisoning* the very session that runs the long task.

## The pattern

The Handoff Pattern lets a long-running skill **interrupt itself and resume on a fresh session**, transferring its in-progress state via a small file rather than via context.

### Three components

**1. A gatekeeper check at the start**

Before doing any real work, ask the user one question: is your context healthy?

```
question: "This skill will consume some context. Subsequent /<long-task> needs lots of context to work well. What's your current state?"
options:
  - "Clean / just started"            → continue
  - "Pack a handoff for new session"  → exit early, save state, give resume instructions
  - "Aware but continue anyway"       → continue with condensed responses
```

Critically: **the user can choose to abort before sinking any time in.** This is the single highest-leverage decision in the pattern.

**2. State serialization on interrupt**

When the user picks "pack handoff" (at start OR mid-flow), the skill writes a `handoff.md` (or any structured file) containing:

- The user's original input/intent
- All answers/state collected so far
- A pointer to the "next step" the resumed skill should jump to
- Any side artifacts (e.g. cached scan results) that took non-trivial work to produce and should not be redone

```markdown
# Handoff — interrupted at <ISO timestamp>

## Original task
<user's initial input>

## Collected state
<key/value of everything answered so far>

## Next step
<which phase / question to resume from>

## Cached artifacts
<paths to any expensive intermediate outputs>
```

**3. A `--resume` trigger and skip logic**

The skill exposes a `--resume <name>` trigger. When invoked:

1. Load the handoff file
2. **Skip the gatekeeper check** (we're already on a fresh session by assumption)
3. **Skip every step whose state is already filled in**
4. Jump straight to "next step"
5. Reference cached artifacts as if the skill had produced them just now

The user's experience: they ran `/clear`, then `/<skill> --resume <name>`, and the Q&A picked up at exactly the right question on a clean context window.

---

## When to adopt this pattern

Adopt it if **any** of these are true:

- Your skill performs 5+ rounds of Q&A
- Your skill calls subagents that return non-trivial summaries
- Your skill's output is itself the input to another long-running operation
- Your skill is commonly invoked late in a session after other work

Skip the pattern if your skill is a one-shot transform (e.g. "format this JSON") with predictable, small context use.

---

## Implementation notes

### The gatekeeper question is the load-bearing piece

You can implement everything else and skip the gatekeeper — and the pattern still has value. But the gatekeeper is what saves users from spending 10 minutes on Q&A only to discover their `/goal` runs out of context. It's a 30-second check that prevents 30 minutes of waste.

Always put it first. Always let the user abort.

### Serialize, don't summarize

The temptation is to compress collected state into a brief "summary" before writing the handoff. Resist this. Write structured key/value state that the `--resume` path can mechanically check. Summaries decay; structured state survives roundtrips.

### Naming the draft

Use kebab-case derived from the user's initial input (e.g. "drop checkout speed" → `checkout-speed`). Append `-2`, `-3` if duplicate. This makes `--resume <name>` memorable without the user copy-pasting a UUID.

### The user's two-command workflow

The resume instructions should always be exactly two commands:

```
1. /clear
2. /<skill> --resume <name>
```

Anything more (three steps, optional flags, conditional branches) and users will get the order wrong. Two commands is the maximum complexity humans reliably execute from instruction text.

### What `--resume` should NOT do

- Do not re-run the gatekeeper check (we're already on fresh context by assumption)
- Do not re-run expensive subagent calls if their output is cached in `~/.<skill>/...`
- Do not re-confirm answers the user already gave
- Do not assume the new session has access to the OLD session's working directory state — load it from disk

### Multi-step skills with optional branches

If your skill has branches (e.g. "subjective task only" sub-flow), record the branch decision in the handoff state. `--resume` reads it and continues in the correct branch.

---

## Limitations

- **Skill cannot programmatically spawn the new session.** Claude Code skills run inside the current process; they cannot start a new interactive session and transfer state to it. The user must manually run `/clear` (or open a new terminal) and then `/<skill> --resume <name>`. There is currently no API to automate this.
- **The user might forget to `/clear`.** Then `--resume` runs on a still-strained context, defeating the point. Mitigation: print the two commands prominently and explain *why* `/clear` matters.
- **State drift between sessions.** If files in the working directory change between handoff and resume, cached artifacts may be stale. Skills that care about this should add a freshness check in `--resume`.

---

## Reference implementation

`goal-workflow-designer`'s `goal` skill implements this pattern in full. See:

- `skills/goal/SKILL.md` Step 2 (Phase 0a) for the gatekeeper check
- `skills/goal/SKILL.md` Step 9 for state serialization
- `skills/goal/SKILL.md` Step 8 for the `--resume` skip logic

The handoff file is written to `~/.claude/goals/<draft-name>/handoff.md` and includes both task state and a `Codebase scan done?` flag so resume can skip the expensive scan step if already complete.

---

## Variations

- **Auto-handoff threshold.** If your skill can estimate remaining context (Claude Code does not expose this directly as of 2026-05, but you can ask the user), the gatekeeper can automatically suggest handoff above a threshold rather than asking.
- **Multi-resume.** Long skills with many phases can serialize state at every phase boundary, allowing `--resume <name> --from <phase>` style overrides. Useful for debugging or partial reruns.
- **Cross-machine handoff.** If `handoff.md` is in a synced directory (Dropbox, iCloud, git), users can pause on machine A and resume on machine B. Side effect, not designed for, but works.

---

## Origin

This pattern was developed for this repo's `goal` skill after recognizing that the very skill meant to *improve* `/goal` outputs was systematically *degrading* them by consuming context before `/goal` even started.

If you adopt the pattern in your own skill, attribution is appreciated but not required (MIT license). A short link back to this document helps other skill authors find the pattern.
