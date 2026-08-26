# gretl Package-Building Prompts

Copy-paste prompts — and, soon, a loadable Claude Skill — that help gretl
(GNU Regression, Econometrics and Time-series Library) contributors turn
their own hansl scripts into a proper gretl function package (`.gfn`), using
an LLM to either walk them through gretl's GUI or actually build the file.

## What's here

| File | What it does | Who it's for |
|---|---|---|
| [`prompts/gui-walkthrough.md`](prompts/gui-walkthrough.md) | A paste-ready prompt that has an LLM walk you through building a package step by step, using only gretl's graphical package editor. Replies in your own language and can explain basics like public vs. private functions along the way. | Anyone with a working function, using a plain chat LLM (ChatGPT, Claude.ai, etc.) — no code execution needed. |
| [`prompts/agent-build.md`](prompts/agent-build.md) | A paste-ready prompt for agentic tools with real file/code access (Claude Code, Claude Cowork, ChatGPT with a code interpreter, etc.). The agent writes the source files itself and, where possible, actually runs `gretlcli --makepkg` to produce a real, DTD-validated `.gfn` — falling back to a ready-to-build source bundle if it can't execute gretl itself. | Contributors comfortable letting an agent do the mechanical work. |
| `skills/package-builder/SKILL.md` | *(forthcoming)* A loadable Claude Skill (or `AGENTS.md`-style file) wrapping the two main prompts, so a session can use this capability automatically. | Anyone using a tool that supports loadable skills. |


## Using a prompt

1. Open the relevant file under `prompts/`.
2. Copy everything between the `---` markers.
3. Paste it into your LLM of choice, and attach (or paste) your `.inp`file(s) — your function definitions, and a sample script if you already have one.

## External resources (not vendored in this repo)

The prompts are grounded in gretl's own documentation, which is maintaine and licensed separately (GNU FDL) and isn't copied in here - link to it instead:

- [gretl function packages: a guide for dummies](https://gretl.sourceforge.net/gfnguide/gfn_for_dummies.html)
- [Gretl Function Package Guide (pkgbook.pdf)](https://sourceforge.net/projects/gretl/files/manual/pkgbook.pdf)

## Status

A work in progress, drafted and checked against real gretl packages and real `gretlcli` build/validate run - not just against gretl's docs in the abstract. The `skills/` entry is a placeholder until the Skill vs. `AGENTS.md` question is settled.

## Contributing

Issues and PRs welcome - in particular, corrections grounded in gretl's actual behavior (a build log, a DTD detail, a spec-file edge case) over general polish.

## License

MIT — see [`LICENSE`](LICENSE). Free to use, copy, and modify, for commercial or academic purposes alike.
