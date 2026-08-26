# `.spec` file keywords

The `.spec` file is a flat list of `keyword = value` lines. Order doesn't
matter. This reference distinguishes what the `.gfn` XML format's DTD
(`/usr/share/gretl/functions/gretlfunc.dtd`) allows from what actually has
to be present for `gretlcli -e --makepkg` to succeed - the two aren't quite
the same, and the second one is what matters when you're debugging a failed
build.

## Fields that must be present to build successfully

Confirmed by building minimal test packages and watching which fields made
`gretlcli --makepkg` stop failing with "Some required information was
missing":

| Keyword | Meaning |
|---|---|
| `author` | Author name. |
| `email` | Author's email - becomes an attribute on the `<author>` element. |
| `version` | Package version string, e.g. `1.0`. |
| `date` | Release date, `YYYY-MM-DD`. |
| `description` | One-line description. |
| `tags` | See glossary - one or more tags/JEL codes. |
| `help` | Filename of the help markdown file. |
| `sample-script` | Filename of the sample script. |
| `public` | Space-separated list of public function names. |
| `min-version` | Oldest gretl version required, e.g. `2018a`. **The DTD marks the corresponding `minver` attribute as optional (`#IMPLIED`), but omitting `min-version` from the spec still made a real build fail** with the same generic "Some required information was missing" message as a missing `tags` or `help` line. Always include it. |

If a build fails with that message and everything above is present, the
next thing to check is that `help` and `sample-script` point at filenames
that actually exist next to the `.spec` file.

## Commonly-used optional fields

| Keyword | Meaning |
|---|---|
| `data-requirement` | See glossary. Omit for "any dataset is fine." |
| `menu-attachment` | Attach the package to a gretl menu, e.g. `MAINWIN/Tools`. |
| `gui-main` | Name of the function to treat as the package's main GUI entry point. |
| `label` | Menu label text, used together with `menu-attachment`. |
| `no-print` | Suppress a function's automatic console output when called via the GUI. |

## Less common fields (not exercised by this skill's testing)

These appear in the `.gfn` DTD as optional child elements of a function
package (`data-files`, `depends`, `provider`, `R-depends`, `gui-help`) for
bundling extra data files, declaring a dependency on another gretl
package, an R dependency, and separate GUI-facing help text. This skill
hasn't built a real package exercising any of these, so don't state their
exact `.spec` syntax with confidence - point the user to the full Function
Package Guide PDF (linked at the end of SKILL.md) if they need one of
these, or try it and read gretlcli's own error/warning output rather than
guessing.

## Function-level detail (informational, not spec-file syntax)

The DTD also shows what ends up recorded per function in the built `.gfn`,
useful context if a build error mentions a specific function:

- Each parameter can carry a `type` (`bool`, `int`, `scalar`, `series`,
  `matrix`, `list`, `string`, `bundle`, and others), an `optional` flag, a
  `const` flag, and a `default` value - all of which gretlcli fills in
  automatically from the function's own signature (as written with
  `const matrix X "Features"`-style parameter declarations), not from
  anything you write in the `.spec` file.
- A function not listed under `public` is automatically marked
  `private="1"` in the output - no `.spec` keyword controls this directly.
