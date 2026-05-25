# skill-to-goal

> A Claude Code skill that interrogates you until your `/goal` prompt is precise enough to actually work.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Skill Format](https://img.shields.io/badge/Claude%20Code-skill-purple)](https://code.claude.com/docs/en/skills)
[![Bilingual](https://img.shields.io/badge/docs-EN%20%2B%20繁中-orange)](SKILL.md)

`skill-to-goal` is a Claude Code skill that turns vague task descriptions into precise goal prompts for Claude Code, OpenAI Codex, or Manus Agent's `/goal` feature. It is a **front-end coach** for the `/goal` feature, not a replacement for it.

You feed it `make this project better`. It walks you through a structured Q&A — five elements (outcome, verification, constraint, iteration policy, error handling) for any task, plus a six-step rubric SOP for subjective work — and outputs a goal prompt that will actually keep the agent iterating for hours instead of quitting in three minutes.

---

## Why this exists

In 2026, Claude Code, OpenAI Codex, and Manus Agent shipped a `/goal` feature within weeks of each other. The feature loops an implementer agent and a reviewer agent until a goal is reached. Anthropic's own research showed that without one, LLMs hit **context anxiety** when their context window fills — they shortcut to a wrap-up and stop, even mid-task.

But `/goal` only works when "done" is defined precisely. Given a vague prompt, the AI invents its own definition of "done" and quits early. Given a precise one, it iterates for hours.

The hard part isn't using `/goal`. It's **writing the prompt that goes into `/goal`**.

This skill automates the prompt-writing part. It is opinionated about what a goal prompt must contain and refuses to let you off the hook with vague answers.

---

## What makes this different

A handful of other Claude Code skills do prompt design ([prompt-architect](https://github.com/ckelsoe/prompt-architect), [prompt-improver](https://github.com/ndpvt-web/prompt-improver), [prompt-master](https://github.com/nidhinjs/prompt-master)). They are general-purpose — they will improve any prompt for any use case. `skill-to-goal` is the opposite: laser-focused on the `/goal` execution model.

| Feature | skill-to-goal | Generic prompt skills |
|---|---|---|
| Targets `/goal`'s five-element framework | ✅ | ❌ (general frameworks like CO-STAR, RISEN) |
| Builds a rubric for subjective tasks | ✅ (6-step SOP) | ❌ |
| Context window health gate | ✅ (Phase 0a) | ❌ |
| **Goal-eligibility check (3-light system)** | ✅ (Phase 0c) | ❌ |
| Handoff packaging for context-strained sessions | ✅ (`--resume`) | ❌ |
| Codebase scan integration via subagent | ✅ (isolated context) | ❌ |
| Objective vs subjective task branching | ✅ | ❌ (single flow) |
| Bilingual term parentheticals (EN + 中文) | ✅ | varies |

If you only need to clean up a generic prompt, use one of the others. If you specifically want a goal that will survive long iteration loops in `/goal`, use this one.

---

## Installation

This is a user-level skill, available across all Claude Code projects on your machine.

```bash
git clone https://github.com/dragon375014/skill-to-goal.git ~/.claude/skills/goal
```

Then add the trigger to your `~/.claude/CLAUDE.md`:

```markdown
# goal
- **goal** (`~/.claude/skills/goal/SKILL.md`) - Goal prompt design coach. Trigger: `/goal`
When the user types `/goal`, OR uses natural language indicating they want to design a goal (e.g. "I want to design a goal", "help me write a goal prompt", "我要設計一個 goal"), invoke the Skill tool with `skill: "goal"` before doing anything else.
```

Output goals will be saved under `~/.claude/goals/<name>/` (the skill creates this directory on first use).

---

## Usage

```
/goal <one-line task description>     # full Q&A from scratch
/goal --use <name>                    # load a saved goal
/goal --resume <draft-name>           # resume an interrupted Q&A
```

Natural language also works: typing `I want to design a goal for refactoring the auth module` or `幫我寫個 goal prompt` will trigger the skill.

> ⚠️ `/goal` with no arguments is intentionally unsupported. You must give the skill at least one sentence to start from — vague triggers produce vague Q&A.

### Example: objective task (engineering)

```
You: /goal drop checkout page response time

Skill: ⚠️ Quick context check first — is your current Claude Code session clean,
       or has it been running a while? (context anxiety affects /goal quality)
You:   Clean, just opened.

Skill: Should I scan the codebase first to find relevant files?
You:   Yes.

Skill: [spawns Explore subagent — returns: src/checkout/, validatePayment(),
        vitest configured, lighthouse in devDependencies]

Skill: Task type — objective or subjective?
You:   Objective.

Skill: I have "drop checkout page response time" as the outcome direction.
       What's the specific target? (e.g. "under 0.2s")
You:   Under 0.3s on slow 3G.

Skill: I saw lighthouse in devDependencies — use it for verification?
You:   Yes, lighthouse mobile slow-3G profile.

[... continues for constraint, iteration policy, error handling ...]

Skill: ✅ Goal saved to ~/.claude/goals/checkout-perf/
       Run /goal --use checkout-perf in a fresh session for best results.
```

The resulting `goal.md` is a complete prompt covering all five elements, ready for `/goal`.

### Example: subjective task (writing / design)

For subjective tasks the skill additionally walks you through the **6-step rubric SOP**:

1. Baseline check (do you have a sample AI output?)
2. Collect "frown points" — specific things that annoyed you
3. Group frown points into dimensions
4. For each dimension, write specific `Never do X` rules
5. List diverse positive directions to avoid overfitting
6. Output both `goal.md` and `rubric.md`

The rubric is what lets the `/goal` reviewer agent actually judge taste-based outputs. Without one, the reviewer has nothing to compare against, and the loop degenerates.

See `examples/` for complete sample outputs.

---

## Design principles

These principles are codified in the skill itself and govern every Q&A round:

1. **Interrogate vague answers.** When the user says "better / smoother / more professional", the skill *must* follow up for a measurable or visible standard. Accepting vague answers is a skill failure.
2. **Smart skipping.** Each round, scan everything already collected and skip known elements. No redundant questions.
3. **1–3 questions per round.** Long question lists make users abandon.
4. **Cite source-talk examples.** Show what a good answer looks like, but tell users not to copy verbatim.
5. **Subjective tasks require a rubric.** Without one, the reviewer agent has nothing to judge against.
6. **Context window health above all.** Phase 0a is a gatekeeper. Don't push through to "finish the skill" if the user's context is strained.
7. **Not every task is `/goal`-shaped.** Phase 0c checks for external credentials, info gaps, human-only decisions, and external blocking dependencies before letting you sink time into Q&A. Yellow → list prerequisites and offer handoff. Red → stop and suggest alternatives. Conservative: any single red flag = overall red.
8. **Bilingual terminology.** First mention of any technical term writes `English (中文)`. Helps both Chinese readers and English readers building vocabulary on this topic.

---

## The Handoff pattern

A novel mechanism this skill introduces — see [docs/handoff-pattern.md](docs/handoff-pattern.md) for the standalone writeup that other skills can adopt.

When a user's Claude Code session is already context-strained at skill invocation, finishing the Q&A and then running `/goal` would compound the problem. The skill instead packages the in-progress state into `handoff.md` and tells the user:

```
1. /clear                           (wipe current context)
2. /goal --resume <draft-name>      (resume Q&A on fresh context)
```

The fresh session loads the handoff, skips already-answered phases, and continues from the exact next question. `/goal` ultimately runs against a full-context window, dodging context anxiety entirely.

This pattern generalizes to any long-running skill that meaningfully consumes context. Take it, use it, adapt it.

---

## Source material

The methodology this skill encodes is drawn from:

- Anthropic's research on **context anxiety** and the implementer/reviewer architecture
- Anthropic's **frontend-design** skill case study (the museum website experiment) — the source for the 6-step rubric SOP
- Andrej Karpathy's writing on **evaluation > prompt engineering**
- The community insight that `/goal` succeeds or fails entirely on prompt precision

A transcript of the source talk that synthesizes these threads is in [examples/source-transcript.md](examples/source-transcript.md).

---

## Contributing

PRs and issues welcome. A few principles for contributors:

- **Don't add elements to the five-element framework.** It's stable on purpose. If you think something's missing, open an issue first.
- **Add example goals to `examples/`** if you have interesting outputs from your own use of the skill.
- **Translations welcome.** The skill is currently bilingual EN/繁中. Other-language support means translating the AskUserQuestion prompts and adding a language-detection branch in Step 5.
- **Test before PR.** Run the skill on your own machine and confirm the change works end-to-end.

---

## License

[MIT](LICENSE). Do whatever you want with it; attribution appreciated but not required.

---

## Related

- [Claude Code skills documentation](https://code.claude.com/docs/en/skills)
- [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)
- [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
- [prompt-architect](https://github.com/ckelsoe/prompt-architect) — general prompt structurer
- [prompt-improver](https://github.com/ndpvt-web/prompt-improver) — Aristotelian first-principles prompt design
