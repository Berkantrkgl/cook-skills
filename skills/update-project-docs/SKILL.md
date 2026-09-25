---
name: update-project-docs
description: Update a project's docs after work ships or at the end of a session — decide whether the work earns a record at all, write the ADR or extract the rule, supersede what it replaced, and clean state.md. Assumes the .agents/project_docs/ layout that init-project-docs sets up. Use when a piece of work has just finished, when a session is ending, when the user says something is done or shipped, or when asked to record a decision, write an ADR, or update the docs.
---

# Updating the docs after work ships

This runs constantly; `init-project-docs` set the structure up and runs once.
It assumes that structure:

```
AGENTS.md                  # short: commands, conventions, traps, routing table
.claude/CLAUDE.md          # `@../AGENTS.md` and nothing else, where Claude Code is used
.agents/project_docs/
├── architecture.md        # the mechanism, the invariants, the rules left behind
├── state.md               # what is decided but not built
├── glossary.md            # only if the project has one
└── adr/                   # README.md (the index) + NNNN-slug.md
```

**The docs are written in English** whatever language the conversation is in,
unless the project's `AGENTS.md` says otherwise.

**If `.agents/project_docs/` does not exist, stop.** If the project has docs in
some other shape, say so and offer `init-project-docs` as a migration — never
migrate unasked. If it has none, offer to set them up. Writing into a shape that
was never established is how one file becomes the only file again.

**If the project carries its own docs skill** (a project-level `skills/*docs*` (under `.claude/`, `.agents/` or the like) whose
description says it is that project's copy), use that one instead and stop here.

**Why this is a step rather than a habit.** "Keep the docs tidy" is a rule
projects already have, written in the docs, and it is the rule that fails. It
fails because it names no moment. This is the moment: the end of a piece of
work, and the end of a session.

---

## Step 0 — does this earn a record at all?

**Most work does not, and that is the load-bearing part of this skill.** A
record is earned by work that:

- **leaves a new rule** — something a future reader must not break, or
- **disproves something the docs currently state**, or
- **is a decision whose reversal would be expensive.**

A one-line fix, a rename, a dependency bump, a test added to existing behaviour,
a refactor that changed no contract: **nothing.** Say so and go to Step 4.

**Work that is not finished earns nothing either** — a session ending halfway
through something is the common case. A record of an intention reads exactly
like a record of a fact. Go to Step 4; `state.md` is where unfinished work is
allowed to show.

The failure this prevents is not laziness, it is the opposite. A doc where
everything earned a section grows at the rate work happens, and at a few
thousand words a week it becomes unreadable inside a quarter — at which point
nobody opens it and the effort was worse than none. **The bar is what keeps the
doc worth reading.**

---

## Step 1 — a decision, or a finding?

Test against all three. **All three must hold** for an ADR:

1. **Hard to reverse.** If it is easy to reverse, you will just reverse it.
2. **Surprising without context.** A future reader looks at the code and asks
   "why on earth did they do it this way?"
3. **The result of a real trade-off.** There were genuine alternatives.

A **constraint imposed from outside** counts as a decision even without
alternatives, if it is invisible in the code — an upstream API that answers a
missing period with a zero rather than an error, a vendor limit that shapes the
design, a compliance rule that rules out the obvious approach.

Everything else is a **finding**. A bug post-mortem is a finding. A measurement
is a finding. *"One answer was simply wrong"* has no rejected alternative and is
not a decision.

Getting this wrong in the safe direction is cheap: a finding written as an ADR
is a description sitting in a decision log. Getting it wrong the other way loses
the reason for something expensive — and since there is no archive, a reason
that is not in an ADR is a reason that will be gone.

---

## Step 2 — write it

### A decision → a new ADR

Number it one above the highest in `project_docs/adr/`. Filename
`NNNN-short-slug.md`. Short — a title and a few paragraphs, not a template to
fill in.

Three things it must do:

- **Name the alternative.** An ADR without a rejected option is a description,
  and somebody will propose the rejected thing again in six months. If there was
  no alternative, re-read Step 1.
- **Carry the measurement, if there was one.** A rule with a figure attached
  survives; a rule without one reads as boilerplate and gets reasoned around.
- **Not restate the mechanism.** How it works belongs in `architecture.md`. The
  same sentence is never in both files — and if you find yourself copying, the
  boundary is the thing to fix.

Then add a row to `adr/README.md` under the right heading.

### A finding → extract what survives it, drop the story

There is no archive. The story of the finding — what was tried, in what order,
what it cost — is **not written down anywhere**; it lives in the session and
goes with it. What survives goes to **exactly one** place:

| the finding leaves | where |
|---|---|
| a rule about how the system behaves, or what a figure may claim | `architecture.md` → **The rules the work left behind** |
| something that bites *while working*, not guessable from the code | `AGENTS.md` → **Traps** |
| a constraint from outside the codebase | a new ADR after all |
| a convention for one area of the repo | `<area>/AGENTS.md` |
| a project-specific word that now means something exact | `glossary.md` (create it if this is the first term) |

State it as a rule, not as a story: *"a folded top-N answer is not a lookup"*,
not *"while measuring X we found that…"*. Keep the figure that makes it land,
drop the narrative arc around it.

**A trap goes where you will be standing when it bites**, which is almost never
the architecture doc. That is the whole reason the two sections are different.
And keep each trap to a paragraph: `AGENTS.md` is loaded into every session, so
its length is paid for every time.

---

## Step 3 — supersede what it replaced

This is the step that gets skipped, and skipping it is what produces a document
that has to warn you about itself.

- **A superseded ADR is never edited and never deleted.** Add
  `Status: superseded by NNNN` under the title and **leave the body alone**.
  Write the new one. This is what keeps the directory free of reading order — a
  reader lands on the old record, sees one line, and goes to the right one. It
  is also, now that there is no archive, the only place the reason for an old
  choice survives.
- **Text that is no longer true anywhere else** — a paragraph in
  `architecture.md`, a trap that stopped being one — is **deleted**. Before
  deleting, check the one thing that would make that a loss: if the paragraph
  carried the *reason* for a decision, that reason must already be in an ADR.
  If it is not, write the ADR first, then delete.
- **Say that it is gone.** `project_docs/` is outside version control by
  default, so a delete is a destroy. Name what you removed in your report to the
  user — they are the only backup.
- **An ADR marked `Status: accepted — not built yet`** (written by
  `lets-cook-together` when the decision was made) becomes `Status: accepted`
  once the work it decided has shipped. The status line is the one line of an
  ADR that is ever changed; the body stays as it was. If the work shipped
  *differently* from what the record says, that is not an edit either — write
  the new ADR and mark the old one superseded.
- **Check for a second copy.** If the rule you just changed is stated in two
  files, you have just made one of them wrong. Grep for it before finishing.
- **A term the work renamed or redefined** is changed in `glossary.md` in the
  same pass, and the old word moves to that entry's `_Avoid_` line rather than
  disappearing — somebody will still say it. The glossary holds only
  project-specific terms, one or two sentences each, and no implementation
  detail.

---

## Step 4 — clean `state.md`

Run this even when Step 0 said the work earned nothing, because a gap can close
without a record being earned — and at the end of a session this is usually the
whole of the job.

- A gap row that closed is **deleted**, not struck through. Its reasoning is in
  the ADR by now, which is what makes deleting it safe. A file of crossed-out
  entries is a file nobody reads to the bottom.
- A gap row the work **disproved** is deleted too, and this is worth checking
  every time. Entries written weeks ago are routinely wrong about the code by
  the time anyone acts on them — a limit off by an order of magnitude, a
  capability described as impossible that the API accepts, "a dozen dead
  exports" that turn out to be zero. **Measure the entry before working from
  it**, and if it was wrong, say so rather than quietly fixing the row.
- A new gap the work opened — including work left half-done at the end of a
  session — gets a row stating its **effect**, not its title.
- Sections with an expiry (a launch date, a migration window) are deleted when
  they expire.
- **Never put a figure in `state.md` that will rot** — a test count, a row
  count, a benchmark. Point at the command that produces it.

---

## Step 5 — verify

Run these from the repository root; do not reason about them.

```bash
# 1. every link resolves, relative to the file it is written in
for f in AGENTS.md .agents/project_docs/*.md .agents/project_docs/adr/*.md; do
  d=$(dirname "$f")
  grep -oE '\]\([^)#]*\.md' "$f" | sed 's/^](//' | sort -u | while read l; do
    [ -f "$d/$l" ] || echo "BROKEN in $f: $l"; done
done

# 2. the dateless files are still dateless
grep -cE '\b20[0-9]{2}-[0-9]{2}-[0-9]{2}\b' .agents/project_docs/architecture.md .agents/project_docs/state.md

# 3. the rule you wrote is not stated twice
grep -rn '<a distinctive phrase from the new rule>' --include='*.md' AGENTS.md .agents/

# 4. the weight — AGENTS.md is read every session, architecture.md often
wc -l AGENTS.md .agents/project_docs/architecture.md
```

(2) is the canary. A date-shaped value can be legitimate — a date passed to an
API is data. A `Shipped 2026-…` opener in a dateless file means this ritual was
skipped and that file is on its way back to being the only file.

(4) is a signal, not a limit. When `architecture.md` passes roughly a thousand
lines, or `AGENTS.md` a few hundred, the usual cause is stories that were kept
where only rules should be — re-read the largest section against Step 2 before
adding to it, and tell the user the file is growing.

If the project defines its own checks for the docs ritual in `AGENTS.md` (a
translation check, a link check of its own), run those too.

---

## What not to do

- **Do not write a section about a piece of work.** That is the habit that
  produces a work log wearing a design doc's name. Take the rule, drop the
  narrative.
- **Do not annotate the directory tree with reasoning.** A structure listing
  that explains *why* is a second copy of the architecture doc, and it drifts
  first because nobody thinks of it as documentation.
- **Do not fold two decisions into one ADR** because they shipped the same day.
  They will be superseded separately, and then neither can be.
- **Do not duplicate reference material that has a machine-readable source.**
  The token file, the schema, the message catalogue, the API types. A markdown
  copy drifts silently, and the drift is discovered by someone acting on it.
- **Do not update the docs for work that is not finished**, beyond its row in
  `state.md`.
- **Do not create an archive file** to avoid deleting. Superseded decisions
  already survive as superseded ADRs; everything else that stops being true is
  meant to go.
