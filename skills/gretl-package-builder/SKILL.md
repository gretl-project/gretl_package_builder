---
name: gretl-package-builder
description: Build a ready-to-install gretl (open-source econometrics software) function package - a .gfn file - out of hansl function code. Covers gathering package metadata, writing the .inp/.spec/sample-script/help-file set, running gretlcli --makepkg to build and DTD-validate the .gfn, and falling back to a source bundle with build instructions when gretlcli isn't available. Use this whenever the user wants to package, publish, or distribute gretl/hansl functions; mentions a ".gfn file", "gretl package", "hansl function package", "gretlcli --makepkg", or the "Function Package Guide"; or shares one or more .inp files of function definitions and wants them turned into an installable package - even if they never use the word "package" explicitly.
---

# Gretl Package Builder

Turn a set of hansl functions into a gretl function package: a single `.gfn`
file (or a zip, if it needs to carry a PDF or extra data files) that anyone
running gretl can install and call. This is a real, mechanical build process
- do it yourself with real tool calls rather than describing the steps.

**Don't fabricate command output.** Only report a command's output if you
actually ran it just now. If you have no shell/code-execution access at all,
say so plainly and go straight to the fallback path (Step 3b).

**Reply in the language the user is writing in.** If that isn't clear, ask.

## Overview of the deliverable

For a package called `mypkg`, you are producing:

- `mypkg.inp` - the function code only (public and helper functions together)
- `mypkg.spec` - metadata and build instructions
- `mypkg_sample.inp` - a short demo script
- `mypkg_help.md` - help text in gretl's markdown subset
- `mypkg.gfn` - the actual built, installable package (if gretlcli is available)
- `Makefile` (optional) - lets the user rebuild without you

See `references/worked-example.md` for a complete, minimal set of these five
files together, and `references/glossary.md` if you need to explain any of
the terms below to the user in plain language.

## Step 0 - Gather what's missing

You need: a package name; which function(s) are public vs.
private/helper; an author name and an email the user actually checks; a
one-line description; one or two tags (conventionally JEL codes, e.g. `C13`,
though the field itself is free text); a minimum gretl version; a data
requirement (or none); and whether a menu attachment or extra bundled data
files are needed (which would make it a zip package instead of a plain
`.gfn`).

Ask for whatever is still ambiguous in one batch, not one question at a
time. But don't block on fields you can reasonably infer yourself - say what
you assumed and let the user correct it:

- **Public vs. private** is usually obvious from the code: if the sample
  script (or the user's description of how the package will be used) calls a
  function directly, it's public. gretlcli infers "private" automatically
  for anything not listed under `public` - see `references/spec-keywords.md`.
- **Data requirement**: if the functions only touch their own arguments
  (matrices, scalars, strings, bundles) and never read a `series` or the
  currently open dataset, that's `no-data-ok`. If they clearly need
  time-series, quarterly/monthly, or panel structure, use the matching
  keyword. Otherwise just omit the field - default gretl behavior is "any
  dataset is fine."
- **Minimum version**: default to something conservative (`2018a` is a safe
  default for typical hansl syntax) unless the user's code uses a construct
  you know is newer. Tell the user what you picked and why you didn't ask.
- **Menu attachment / zip**: default to no menu attachment and a plain
  `.gfn` unless the user mentions a PDF help file, extra data files, or
  wanting the package to appear in one of gretl's own menus.

## Step 1 - Check what your environment can do

- Try `gretlcli --version`.
- If it's missing and you have real shell, package-manager, and network
  access, try installing it (Debian/Ubuntu: `apt-get update && apt-get
  install -y gretl gretl-common gretl-data`). If that's not how your
  environment works, or the install fails, don't keep retrying with
  guesses - note that `gretlcli` isn't available and move to Step 3b.
- Tell the user once, plainly, which path you're taking and why. You don't
  need to narrate every intermediate command.

## Step 2 - Write the source files

Copy the user's actual function code into `mypkg.inp` **verbatim** - don't
rewrite, "clean up", or "improve" their logic. You can't run their real use
cases to check a rewrite is equivalent, so an unrequested change risks
silently breaking something that worked. If a combined file mixes function
code and a demo script together, split it the way `references/worked-example.md`
does - function definitions in `mypkg.inp`, everything that calls them in
`mypkg_sample.inp`.

For the `.spec` file, use only the keywords that actually apply - see
`references/spec-keywords.md` for the full list, which ones are actually
required to build (this differs slightly from what the DTD alone would
suggest), and what each one means.

For `mypkg_sample.inp`: start with `include mypkg.gfn`; run with no dataset
already open unless the demo genuinely needs one, in which case prefer a
dataset bundled with gretl; comment out any alternative example call that
needs different data, so the script still runs clean as-is; keep it well
under a minute to run.

For `mypkg_help.md`: gretl's supported markdown subset is `#`/`##`
headings, `**bold**`, `*italic*`/`_italic_`, `` `monospace` ``, `- ` for
itemized lists, `1. ` for numbered lists, and triple-backtick fenced code
blocks. No nested lists, and always leave a blank line before any list or
code block.

For the `Makefile`, copy `assets/Makefile.template` and substitute the
package name - the indent before `gretlcli` must be a literal tab
character, not spaces, or `make` will refuse to run it.

## Step 3a - If `gretlcli` is available, build it yourself

Run `gretlcli -e --makepkg mypkg.inp` for real, then:

- Confirm the output actually contains `mypkg.gfn: validated against DTD
  OK` (or the equivalent failure message) - don't assume success just
  because the command didn't crash.
- Compare the gretl version you're running against the user's stated
  minimum version. If theirs is newer than what you have, say so plainly: a
  successful build here doesn't prove the package runs on the version they
  claimed to need.
- If the build fails, show the actual error text and fix what you
  reasonably can. Common causes: a hansl syntax error; a missing required
  spec field (check `references/spec-keywords.md` - a missing `tags`,
  `help`, or `min-version` line is the usual culprit, and gretlcli's error
  for all of these is just the generic "Some required information was
  missing"); or a spec file pointing to the wrong filename for the
  help/sample-script files. Retry, but don't loop silently more than a
  couple of times before showing the user what's wrong.
- Give the user the resulting `.gfn` file plus the source files as a
  download.
- Offer - but don't insist - to also run the sample script as a further
  smoke test, and warn first if that would need something the environment
  might lack (network access for a `depends` package, or a dataset that
  isn't bundled with gretl).

## Step 3b - If `gretlcli` isn't available

Package the source files from Step 2 (including the Makefile) into a zip
with this layout:

```
mypkg/
└── src/
    ├── mypkg.inp
    ├── mypkg.spec
    ├── mypkg_sample.inp
    ├── mypkg_help.md
    └── Makefile
```

Tell the user plainly that you couldn't build it yourself and why, then
give them the exact commands to run on their own machine from inside the
`src` directory: `gretlcli -e --makepkg mypkg.inp`, or just `make` if they
have it installed.

## Throughout

Be ready to explain gretl packaging concepts in plain language whenever
asked - see `references/glossary.md` - keep it short, not lecture-length.

At the end of your first full answer, give the user these two links in
case they want to go deeper or you can't fully resolve something:

- https://gretl.sourceforge.net/gfnguide/gfn_for_dummies.html
- https://sourceforge.net/projects/gretl/files/manual/pkgbook.pdf (the full
  Function Package Guide - the authoritative source for less common fields
  like `depends`, `data-files`, and `menu-attachment` that this skill
  doesn't cover in depth)

## Where this skill's facts come from

- `/usr/share/gretl/functions/gretlfunc.dtd`, shipped with the `gretl`
  package itself, defines exactly which elements/attributes a built `.gfn`
  must and may contain - this is the ground truth for what's required vs.
  optional.
- Several details in `references/spec-keywords.md` (notably: `min-version`
  is effectively mandatory for a successful build, and the public/private
  split needs no separate `private` keyword) were confirmed by actually
  building minimal test packages with `gretlcli 2023c --makepkg` and
  observing what made the build succeed or fail with "Some required
  information was missing" - not just read off the DTD or guessed.
- https://gretl.sourceforge.net/gfnguide/gfn_for_dummies.html - the
  official beginner's guide, fetched and cross-checked against the above.
- https://sourceforge.net/projects/gretl/files/manual/pkgbook.pdf - the
  full Function Package Guide; not machine-readable by the tools used to
  build this skill, so treat it as the deeper source for anything this
  skill is vague about, rather than assuming this skill fully reflects it.
