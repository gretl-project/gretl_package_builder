# Paste-ready prompt: let an agentic LLM actually build your gretl package

**For gretl maintainers:** this is the third variant, distinct from the other two.
Where the earlier prompts walk a *person* through the GUI, this one is written for an
*agent* with real file and code-execution tools (Claude Code, Claude Cowork, Claude.ai
with code execution enabled, ChatGPT with a code interpreter, or similar) — it tells
the agent to do the mechanical work itself rather than describe it.

This is grounded in direct testing, not just the docs: `gretl` installs via `apt-get`
on a standard Ubuntu sandbox, and running `gretlcli -e --makepkg` against a real
published package's source files (`knn`) reproduced the official `.gfn` file
byte-for-byte (aside from one cosmetic line). "Cloud" here just means the agent's own
sandbox — there's no separate gretl-as-a-service API — so the prompt collapses
"local" and "cloud" into one case: whatever execution environment the agent has.

---

## Copy everything between the lines below and paste it into your agentic tool of choice

---

This is a real task — please act on it, don't just describe or discuss how you'd
approach it. If you have file and/or code-execution tools available in this
conversation, use them now rather than asking my permission to check.

You are helping me build a real gretl (open-source econometrics package) function
package end-to-end: a `.gfn` file. gretl is the econometrics package; hansl is its
scripting language. Do as much of the mechanical work yourself as your environment
allows, rather than just walking me through manual steps.

**Important — don't fabricate tool output.** Only report a command's output if you
actually ran it through a real tool call just now. If you have no code-execution or
shell access at all in this conversation, tell me that plainly up front and go
straight to the fallback path (Step 3b below) — never simulate or invent a plausible-
looking success message like "validated against DTD OK" unless you truly produced it.

**Language:** reply entirely in the same language I use when I write to you. If that
isn't clear, ask me which language I'd like, then continue in that language.

### Quick glossary (use this to answer my basic questions accurately)

- **Public function** — listed in a package's `public` keyword; directly callable by
  anyone who installs the package.
- **Private / helper function** — loaded but not listed under `public`; still shipped
  with the package, callable internally, but not exposed to package users.
- **`data-requirement`** — omit entirely if the package just needs any old dataset;
  otherwise one of `no-data-ok`, `needs-time-series-data`, `needs-qm-data`, or
  `needs-panel-data`.
- **`tags`** — one or more space-separated JEL classification codes, e.g. `C13`.
- **`min-version`** — the oldest gretl version the package should require, in
  `<year><letter>` form, e.g. `2018a`.
- **Help text** — plain text/Markdown or a PDF (one or the other, not both).
- **Sample script** — a short demo that must start with `include <pkg>.gfn`, run with
  no dataset already open, and finish in well under a minute.
- **Menu attachment** — optional; lets the package add itself to one of gretl's own
  menus (`MAINWIN/...` or `MODELWIN/...`).
- **Simple package vs. zip package** — plain `.gfn` unless it needs to carry a PDF or
  extra data files, in which case it becomes a zip.

### What I'm giving you

*(This part is a note to you, the person pasting this prompt — not to the AI: attach
your own files here before you send this message.)*

I'm attaching my code as file(s): a function-definition `.inp` file, and — if I
already have one — a sample-script `.inp` file. If I've only got one combined file
with both mixed together, split it yourself into the two, the same way the example
below is split.

**[Attach your files now: your function-definition `.inp` file, and your
sample-script `.inp` file too if you have one.]**

### Step 0 — Fill in what's missing

If you don't already have all of the following, ask me for just what's missing, in
one batch: package name; which function(s) are public vs. private/helper; my name and
an email address I check; a one-line description; one or two JEL tags; a minimum
gretl version (default to something conservative unless my code needs newer syntax);
a data requirement (or none, if any dataset is fine); and whether I want a menu
attachment or need to bundle extra data files (which would mean a zip package).

### Step 1 — Check what your own environment can do

- Try running `gretlcli --version`. If it responds, note the version number and skip
  the rest of this step.
- If it's not found, work out what kind of environment you're actually in:
  - A **disposable sandbox** (e.g. code execution built into a browser-based chat
    interface) that resets and disappears after this session, never touching my real
    computer — if that's you, just try installing gretl yourself without asking (on a
    Debian/Ubuntu-type sandbox: `apt-get update && apt-get install -y gretl
    gretl-common gretl-data`). Nothing persists, so there's nothing to confirm.
  - **My actual local machine** (e.g. a tool like Claude Code running as a process on
    my own computer) — if that's you, ask me before installing anything system-wide.
    That's a real, permanent change to my machine, not a disposable container.
  - If you genuinely have more than one execution environment available to you (for
    instance, both my local machine and a separate sandbox), ask me which one I'd
    like you to use before doing anything.
  - If you can't tell which situation you're in, say so and ask me rather than
    guessing.
- Either way: if I say no, or the install fails, don't keep retrying with guesses —
  just note that `gretlcli` isn't available and move to Step 3b.
- Tell me once, plainly, which situation you're in and what you're doing about it —
  you don't need to narrate every intermediate command.

### Step 2 — Build the source files

Using my function code and everything gathered in Step 0, write these files, named
after my package (say it's called `mypkg`):

- **`mypkg.inp`** — the function code only (public and helper functions together;
  nothing else).
- **`mypkg.spec`** — metadata and build instructions. Use only the keywords that
  actually apply; don't pad the file with irrelevant ones.
- **`mypkg_sample.inp`** — the sample script. Must start with `include mypkg.gfn`. If
  the demo needs a dataset, prefer one bundled with gretl; if any alternative example
  call in the script needs different data, comment it out so the script still runs
  clean as-is.
- **`mypkg_help.md`** — help text in gretl's supported markdown subset: `#`/`##`
  headings, `**bold**`, `*italic*` or `_italic_`, `` `monospace` ``, `- ` for
  itemized lists, `1. ` for numbered lists, and fenced code blocks with three
  backticks. No nested lists, and a blank line before any list or code block.
- **`Makefile`** (optional, but include it if you're handing me the source bundle to
  build myself):
  ```
  PKG = mypkg
  $(PKG).gfn: $(PKG).inp $(PKG).spec $(PKG)_sample.inp $(PKG)_help.md
  	gretlcli -e --makepkg $(PKG).inp
  ```
  (that indent before `gretlcli` must be a literal tab character, not spaces).

**Worked example**, showing the full set together (a tiny function that computes
percentage change):

`pcchange.inp`:
```hansl
function series pc(series y "Series to process")
    series ret = 100 * diff(y)/y(-1)
    return ret
end function
```

`pcchange.spec`:
```
author = A. U. Thor
email = author@somewhere.net
version = 1.0
date = 2025-01-01
description = Percentage change of a series
tags = C10
min-version = 2018a
public = pc
help = pcchange_help.md
sample-script = pcchange_sample.inp
```

`pcchange_sample.inp`:
```hansl
include pcchange.gfn
open denmark.gdt
series pcLRM = pc(LRM)
print LRM pcLRM --byobs
```

`pcchange_help.md`:
```markdown
# pcchange

Computes the percentage change of a series.

**pc(y)**

- `y`: *series*, the series to process.

Returns the percentage change of `y`, as a series.
```

### Step 3a — If `gretlcli` is available, build it yourself

Run `gretlcli -e --makepkg mypkg.inp` for real, then:

- Confirm the output actually contains `mypkg.gfn: validated against DTD OK` (or the
  equivalent failure message) — don't assume success just because the command didn't
  crash.
- Compare the gretl version you're running against my stated minimum version. If mine
  is newer than what you have, tell me plainly: a successful build here doesn't prove
  the package will run correctly on the version I claimed to need.
- If the build fails, show me the actual error text and fix what you reasonably can
  (common causes: a hansl syntax error, a missing required spec field, or a spec file
  pointing to the wrong filename for the help/sample-script files) — then retry. Don't
  loop silently more than a couple of times before showing me what's wrong.
- Give me the resulting `.gfn` file plus the source files as a download.
- Offer — but don't insist — to also run the sample script itself as a further smoke
  test, and warn me first if that would need something your environment might lack
  (network access for a `depends` package, or a dataset that isn't bundled).

### Step 3b — If `gretlcli` isn't available to you

Package the source files from Step 2 (including the Makefile) into a zip with this
layout:

```
mypkg/
└── src/
    ├── mypkg.inp
    ├── mypkg.spec
    ├── mypkg_sample.inp
    ├── mypkg_help.md
    └── Makefile
```

Tell me plainly that you couldn't build it yourself and why, then give me the exact
commands to run on my own machine from inside the `src` directory:

```
gretlcli -e --makepkg mypkg.inp
```

or, if I have `make` installed:

```
make
```

### Throughout

Be ready to explain gretl packaging concepts in plain language whenever I ask (see
the glossary above) — keep it short, not lecture-length.

At the end of your first full answer, give me these two links in case I want to go
deeper or you can't fully resolve something:
- https://gretl.sourceforge.net/gfnguide/gfn_for_dummies.html
- https://sourceforge.net/projects/gretl/files/manual/pkgbook.pdf

---
