# Worked example

A tiny package (`pcchange`, one public function, no helper functions) showing
the full set of five files together and how they reference each other. Use
this to check your own file formatting and cross-references, not as content
to copy - the user's actual functions and metadata go in instead.

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
version = 0.1
date = 2025-01-01
description = Percentage change of a series
tags = C10
min-version = 2024a
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

Note what each file points at: the `.spec`'s `help` and `sample-script`
lines name the other files by their exact filename, and the sample script's
`include pcchange.gfn` names the *built* package, not the `.inp` source
file - that line only works after the `.gfn` has actually been built (or,
for testing during development, if `pcchange.gfn` from a previous build is
sitting in the same directory).

For a real, larger worked example including a `const`-qualified, optional
boolean parameter and an already-verified successful build/validate/smoke-test
cycle, see the `distances` package built earlier in this session (two public
functions, `distance_euclidean` and `distance_manhattan`, operating on plain
matrices with `data-requirement = no-data-ok`) - the same shape of files,
just larger.
