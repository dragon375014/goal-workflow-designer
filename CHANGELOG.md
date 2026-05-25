# Changelog

All notable changes to `skill-to-goal` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
