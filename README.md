# gretl Package-Building Prompts

Copy-paste prompts — and now a loadable Claude Skill — that help gretl
(GNU Regression, Econometrics and Time-series Library) contributors turn
their own hansl scripts into a proper gretl function package (`.gfn`), using
an LLM to either walk them through gretl's GUI or actually build the file.

## What's here

| File | What it does | Who it's for |
|---|---|---|
| [`prompts/gui-walkthrough.md`](prompts/gui-walkthrough.md) | A paste-ready prompt that has an LLM walk you through building a package step by step, using only gretl's graphical package editor. Replies in your own language and can explain basics like public vs. private functions along the way. | Anyone with a working function, using a plain chat LLM (ChatGPT, Claude.ai, etc.) — no code execution needed. |
| [`prompts/agent-build.md`](prompts/agent-build.md) | A paste-ready prompt for agentic tools with real file/code access (Claude Code, Claude Cowork, ChatGPT with a code interpreter, etc.). The agent writes the source files itself and, where possible, actually runs `gretlcli --makepkg` to produce a real, DTD-validated `.gfn` — falling back to a ready-to-build source bundle if it can't execute gretl itself. | Contributors comfortable letting an agent do the mechanical work. |
| [`skills/gretl-package-builder/SKILL.md`](skills/gretl-package-builder/SKILL.md) | A loadable Claude Skill wrapping the agent-build workflow above, so a Claude session picks up the whole process automatically — no copy-pasting a prompt. Packaged, drag-and-drop-installable version at [`skills/gretl-package-builder.skill`](skills/gretl-package-builder.skill). | Anyone using a Claude product that supports Skills (claude.ai, Claude Code, Cowork). |

## Using a prompt

1. Open the relevant file under `prompts/`.
2. Copy everything between the `---` markers.
3. Paste it into your LLM of choice, and attach (or paste) your `.inp` file(s) — your function definitions, and a sample script if you already have one.

## The (Claude) Skill

`skills/gretl-package-builder/` packages the same agent-build workflow as
`prompts/agent-build.md`, but as a [Claude Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) —
a small folder of instructions plus reference material that Claude loads
automatically when it's relevant, instead of you copy-pasting a prompt
every time. It covers the full build: gathering package metadata, writing
the `.inp`/`.spec`/sample-script/help-file set, running `gretlcli
--makepkg` to build and DTD-validate the `.gfn`, and falling back to a
ready-to-build source bundle when `gretlcli` isn't available in the
environment.

### Installing it in claude.ai

1. Grab [`skills/gretl-package-builder.skill`](skills/gretl-package-builder.skill) from this repo (it's just a zip of the `skills/gretl-package-builder/` folder — nothing extra happens at packaging time).
2. In claude.ai, make sure **Code execution and file creation** is turned on — Skills run inside Claude's sandboxed environment, so this is a prerequisite, not optional.
3. Find the Skills section in **Settings** (Anthropic has it under **Capabilities → Skills** as of this writing, though the exact menu path has moved before — if it's not where you expect, search Settings for "Skills") and upload the `.skill` file there.
4. That's it — no further setup. Claude will pull the skill in on its own the next time you ask it to build, package, or distribute a gretl/hansl function package.

Custom Skills currently require a Pro, Max, Team, or Enterprise plan, and
are per-user (not centrally pushed to a whole org) — see Anthropic's own
[Creating custom Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)
article for the current, authoritative steps if anything above looks out
of date.

### Installing it in opencode

[opencode](https://opencode.ai) reads the same kind of `SKILL.md` folder directly — no separate packaging step, and no zip support, so use the unpacked `gretl-package-builder` folder from this repo's `skills/` directory, not the `.skill` file.

Copy that folder into `~/.config/opencode/skills/` in your home directory (on Windows probably, `%USERPROFILE%\.config\opencode\skills\`) so it looks like this:

```
~/.config/opencode/
└── skills/
    └── gretl-package-builder/
        ├── SKILL.md
        ├── references/
        └── assets/
```

That folder is normally hidden from view, so on macOS use Finder's
`Cmd + Shift + G` ("Go to Folder") and on Windows type the path into File
Explorer's address bar to get there. Create any of the `.config`,
`opencode`, or `skills` folders yourself if they don't exist yet.

Restart opencode. Either opencode picks the skill up automatically, and test it by typing:

```
/gretl-package-builder Are you there?
```

You may need to initialize it first, via the `/Skills` command in the opencode console.


### Contributing to the skill

The source of truth is the unpacked folder, `skills/gretl-package-builder/`
(`SKILL.md` plus `references/` and `assets/`) — edit that, not the `.skill`
zip directly, and regenerate the zip afterward so the two don't drift apart.
`skill-creator` is Anthropic's own meta-skill for authoring and packaging
skills — it's what built this one in the first place, inside a Claude
Cowork session. It isn't something with a confirmed standalone repo you
can `git clone` onto your own machine independent of a Claude session. If a Claude Code or Cowork session you're working in happens to have `skill-creator` available as a skill, you can ask Claude to rebuild the package for you — it's worth doing when you have it, since it also validates `SKILL.md`'s frontmatter before packaging:

```python
python -m scripts.package_skill skills/gretl-package-builder
```

run from wherever that session's `skill-creator/scripts/` directory lives,
pointing at this repo's `skills/gretl-package-builder` folder.

Otherwise — and this is the dependable, no-dependency path — a `.skill`
file is just a zip of the skill folder with the skill's own directory name
as the root and its `evals/` subfolder left out, so you can build one by
hand from this repo's `skills/` directory:

```bash
cd skills
zip -rD gretl-package-builder.skill gretl-package-builder \
  -x "gretl-package-builder/evals/*" -x "*.DS_Store" -x "*__pycache__*"
```

Either way you end up with a fresh `gretl-package-builder.skill` in
`skills/`, overwriting the old one — just double-check `SKILL.md`'s YAML
frontmatter by eye first if you used the plain-zip path, since that route
skips the validation step the script does for you. Please
keep any behavioral claims in `SKILL.md` and `references/` grounded in an
actual `gretlcli` build/validate run rather than the docs alone (see
`references/spec-keywords.md` for the kind of build-log-verified detail
we're aiming for) — this repo already caught at least one place where the
official DTD and gretlcli's real behavior at build time don't quite agree
(`min-version` is optional per the DTD but effectively mandatory for a
successful build).

## External resources (not vendored in this repo)

The prompts are grounded in gretl's own documentation, which is maintained
and licensed separately (GNU FDL) and isn't copied in here - link to it
instead:

- [gretl function packages: a guide for dummies](https://gretl.sourceforge.net/gfnguide/gfn_for_dummies.html)
- [Gretl Function Package Guide (pkgbook.pdf)](https://sourceforge.net/projects/gretl/files/manual/pkgbook.pdf)

## Status

A work in progress, drafted and checked against real gretl packages and
real `gretlcli` build/validate runs - not just against gretl's docs in the
abstract. The Skill has been through one round of with-skill-vs-baseline
testing (see the eval notes in `skills/gretl-package-builder/evals/`); more
real-world use will keep shaking out edge cases.

## Contributing

Issues and PRs welcome - in particular, corrections grounded in gretl's
actual behavior (a build log, a DTD detail, a spec-file edge case) over
general polish.

## License

MIT — see [`LICENSE`](LICENSE). Free to use, copy, and modify, for
commercial or academic purposes alike.