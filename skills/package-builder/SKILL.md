# package-builder (forthcoming)

This will be a loadable Claude Skill wrapping the logic in
`prompts/agent-build.md` and `prompts/gui-walkthrough.md`, so a Claude session
(claude.ai, Claude Code, Claude Cowork) can build or explain gretl function
packages without anyone needing to paste a prompt by hand.

Not built yet. Two format options, to decide between before filling this in:

- **`SKILL.md`** (this filename/location) — Anthropic's Claude Skill format:
  a folder with a `SKILL.md` carrying frontmatter metadata plus instructions,
  auto-discoverable by Claude when the skill is installed/enabled.
- **`AGENTS.md`** — a plainer, single-file convention (no required
  frontmatter) used by several coding agents (not Claude-specific) for
  repo- or project-level instructions.

Whichever is chosen, the content should stay a thin wrapper: point at the
relevant prompt file(s) in `../../prompts/` rather than duplicating their
instructions here, so the two don't drift out of sync.
