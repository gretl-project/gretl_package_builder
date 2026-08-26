# Glossary

Plain-language definitions for explaining gretl packaging concepts to a
user. Keep explanations short when using these - this is reference
material, not a script to read aloud.

**Public function** - listed in a package's `public` keyword; directly
callable by anyone who installs the package.

**Private / helper function** - shipped with the package and callable
internally (e.g. one public function calling a helper), but not listed
under `public`, so package users can't call it directly. No separate
keyword is needed for this: gretlcli treats any function defined in the
`.inp` file but not named in `public` as private automatically. Confirmed
by building a two-function test package with only one name listed under
`public` - the other came back tagged `private="1"` in the resulting `.gfn`
with no extra spec line needed.

**`data-requirement`** - omit entirely if the package just needs any old
dataset (or no dataset at all is being *checked for* - see the no-data-ok
case below); otherwise one of:
- `no-data-ok` - the functions don't touch a dataset or `series` at all
  (e.g. they only operate on matrices/scalars/bundles that are passed in
  directly, as in a distance-matrix or matrix-algebra package).
- `needs-time-series-data`
- `needs-qm-data` (quarterly or monthly data)
- `needs-panel-data`

**`tags`** - one or more tags describing the package's subject area. By
convention these are JEL classification codes (e.g. `C13`), and that's what
this skill defaults to, but the underlying field is free text (gretl's own
GUI wizard shows plain category names like "General" or "Univariate
Time-Series Models" as examples too) - so a code or a short phrase both
work.

**`min-version`** - the oldest gretl version the package should require, in
`<year><letter>` form (e.g. `2018a`). This field is technically optional in
the `.gfn` file's own DTD, but in practice `gretlcli --makepkg` refuses to
build without it - see `references/spec-keywords.md`.

**Help text** - plain text/Markdown or a PDF (one or the other, not both).

**Sample script** - a short demo that must start with `include <pkg>.gfn`,
run with no dataset already open (unless it genuinely needs one), and
finish in well under a minute.

**Menu attachment** - optional; lets the package add itself to one of
gretl's own menus (`MAINWIN/...` or `MODELWIN/...`).

**Simple package vs. zip package** - a plain `.gfn` file is enough unless
the package needs to carry a PDF help file or extra data files, in which
case it becomes a zip archive instead.
