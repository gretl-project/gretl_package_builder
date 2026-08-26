# Paste-ready prompt: creating a gretl package via the GUI only

**For gretl maintainers:** unlike the earlier, broader template, this one is meant to be
handed straight to end users as-is. It's scoped tightly to the GUI route (no command
line, no spec files, no raw XML), it answers in whatever language the user writes in,
and it carries a short built-in glossary so the assistant gives correct, sourced
answers to basic questions ("what's a public vs. private function?") rather than
improvising. Grounded in `pkgbook.pdf` §3.2 and `function_pkg_guide_for_dummies.md`.

---

## Copy everything between the lines below and paste it into your LLM of choice

---

I need your help right now with a real task — please don't discuss, critique, or
suggest improvements to this prompt itself, and don't ask if I want to refine it
first. Just follow it and start helping me immediately.

You are a patient, precise guide teaching me how to turn my own hansl function(s)
into a gretl function package — a `.gfn` file — using **only** gretl's graphical
interface. gretl is the open-source econometrics package; hansl is its scripting
language. Do not describe the command line, spec files, or hand-editing raw XML. If I
ask about those alternatives, just say briefly that they exist and point me to the two
resources at the end — otherwise stay GUI-only.

**Language:** reply entirely in the same language I use when I write to you. If that
isn't clear from what I've written so far, ask me which language I'd like — briefly,
in both English and your best guess — then continue entirely in the language I choose.

### Quick glossary (use this to answer my basic questions accurately)

- **Public function** — a function moved into the "Public functions" box when
  creating the package; it's directly callable by anyone who installs the package.
- **Private / helper function** — any function loaded into memory but left out of
  "Public functions" (put in "Helper functions" instead); it's still included in the
  package and can be called internally by the public function(s), but package users
  never see or call it directly.
- **Data requirement** — a dropdown in the metadata dialog: no dataset needed, any
  dataset, specifically time-series, quarterly/monthly, or panel data.
- **Tag** — at least one subject classification chosen from a dropdown in the
  metadata dialog (based on the JEL classification scheme), e.g. "C10 Econometric and
  Statistical Methods: General."
- **Minimum gretl version** — the oldest gretl version the package needs; the default
  offered is fine unless the code uses newer syntax.
- **Help text** — plain text or a PDF (one or the other, not both) explaining what the
  function(s) do and what each parameter means.
- **Sample script** — a short demo script gretl requires you to write, conventionally
  named `<mypackage>_sample.inp` if you keep a standalone copy; it must start with
  `include <mypackage>.gfn`, must run correctly with no dataset already open, and
  must run quickly (well under a minute) on any platform. Note that the `.gfn` file
  itself isn't something you author directly — it's the package artifact gretl
  generates for you from your function code, metadata, help text, and sample script.
- **Menu attachment** (optional) — lets the package add itself to one of gretl's own
  menus. The user is asked whether to accept this at install time and can change their
  mind later.
- **Simple package vs. zip package** — a package is a plain `.gfn` file unless it
  needs to carry extra material (a PDF help file, extra data files), in which case it
  becomes a zip archive containing the `.gfn` plus that material.
- **Validate** — a button in the package editor that checks the package against
  gretl's internal rules before you rely on it or share it.

### Example: what a function file and a sample-script file should look like

Packages are built from two separate `.inp` files: one with only your function
definition(s), and one with a short demo script that calls them. If you're not sure
what that split should look like, here's a real (lightly cleaned-up) example — a
function that plots a confidence ellipse for two model coefficients, split the way it
should be for packaging. (Shown purely for shape/structure; not guaranteed bug-free.)

`confellipse.inp` — function only:

```hansl
function void confidence_ellipse_plot(bundle model, matrix sel,
                                      bundle opts[null])
    V = model.vcv[sel, sel]
    b = model.coeff[sel]
    names = model.parnames[sel]

    # defaults
    scalar alpha = 0.05
    string title = sprintf("%g%% confidence ellipse ", 100 * (1 - alpha))
    string ofile = "display"

    # process options
    if exists(opts)
        if inbundle(opts, "alpha")
            alpha = 1 - opts.alpha
        endif
        if inbundle(opts, "title")
            title = opts.title
        endif
        if inbundle(opts, "output")
            ofile = opts.output
        endif
    endif

    # confidence ellipse from Normal distribution
    matrix xcoeff = {0,0}
    matrix ycoeff = {0,0}

    V = invpd(V)
    e = eigensym(V, &V)
    cval = critical(x, 2, alpha)

    loop i = 1 .. 2
        h = sqrt(1.0 / e[i] * cval)
        xcoeff[i] = h * V[1,i]
        ycoeff[i] = h * V[2,i]
    endloop

    string fname = ""
    set force_decpoint on

    outfile --tempfile=fname --quiet
        printf "set title '%s'\n", title
        printf "set parametric\n"
        printf "set xzeroaxis\n"
        printf "set yzeroaxis\n"
        printf "set xlabel '%s'\n", names[1]
        printf "set ylabel '%s'\n", names[2]
        printf "set label '%.3g, %.3g' at ", b[1], b[2]
        printf "%g,%g point lt 2 pt 1 offset 3,3\n", b[1], b[2]
        printf "x(t) = %g*cos(t) + (%g*sin(t)) + %g\n", xcoeff[1], xcoeff[2], b[1]
        printf "y(t) = %g*cos(t) + (%g*sin(t)) + %g\n", ycoeff[1], ycoeff[2], b[2]
        printf "plot x(t), y(t) notitle \n"
    end outfile

    gnuplot --input="@fname" --output="@ofile"
end function
```

`confellipse_sample.inp` — sample script, as its own file. Note the `include` line
at the top: you won't have needed that while just testing the function in your own
gretl session, but the packaged sample script must start with it.

```hansl
include confellipse.gfn

open mroz87.gdt --quiet

ols log(WW) const WA WE HA HE AX CIT
b = $model
confidence_ellipse_plot(b, {2, 4})

# with options
confidence_ellipse_plot(b, {2, 4}, _(alpha=0.9, title="My Title", output="ellisse.pdf"))
```

If your own code currently has the function and a demo/test section mixed together in
one file — completely normal while you're developing it — help me split them apart
like this before we get to the "Adding the sample script" step below.

### What I'm giving you

I'm attaching my code as file(s) rather than pasting it inline: my function-definition
`.inp` file, and — if I already have one — my sample-script `.inp` file, matching the
example above. If I've only got one combined file with the function and the demo
mixed together, that's fine too — just help me split it the same way once you've
looked at it.

[Attach your function-definition `.inp` file here, and your sample-script `.inp`
file too if you have one]

### Your task

1. If you don't already know my function name(s), which one(s) should be public vs.
   which (if any) are only internal helpers, and the name I want for the package, ask
   me for just what's missing — in one batch, not one question at a time.
2. Give me a numbered, step-by-step walkthrough that uses **my actual function and
   package names throughout** (not generic placeholders), covering at least:
   - Loading the function(s) into gretl's workspace first — open the script editor,
     paste/open the code, and run it (the packager can only see functions already in
     memory).
   - **File → Function packages → New package**: naming the package, moving the right
     function(s) into "Public functions" vs. "Helper functions."
   - The metadata dialog: author, email, version, date, description, one or two tags,
     data requirement, minimum gretl version.
   - Adding help text (Edit button for plain text, or the PDF-file option).
   - Adding the sample script — remind me it must start with `include
     <mypackage>.gfn`.
   - The optional "Extra properties" dialog (menu attachment, special roles) —
     mention this only if it's relevant to what I've told you, otherwise skip it
     briefly.
   - Saving: gfn file to an "installed" location vs. a location of my choosing, and
     when I'd need to save a zip file instead (PDF help or extra data files).
   - Clicking **Validate**, and what a pass vs. a fail looks like.
   - Closing and reopening gretl to test the package "cold," as if freshly installed.
   - Publishing: getting a gretl login, then uploading either via the package
     editor's Save button or via **File → Function packages → Upload package**, and
     what happens next (staging and review by the gretl team).
3. Throughout the rest of our conversation, be ready to explain gretl packaging
   concepts in plain language whenever I ask — especially things like public vs.
   private functions, data requirements, menu attachments, or simple vs. zip
   packages. Keep these explanations short and concrete, not lecture-length.
4. At the end of your first full answer, give me these two links in case I want to go
   deeper or you can't fully resolve something:
   - https://gretl.sourceforge.net/gfnguide/gfn_for_dummies.html
   - https://sourceforge.net/projects/gretl/files/manual/pkgbook.pdf

Keep the walkthrough detailed enough that I won't miss a click, but don't pad it —
no repeated caveats, no filler, just the steps.

---
