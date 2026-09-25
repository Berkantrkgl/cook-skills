---
name: init-project-docs
description: Set up (or migrate to) a durable project documentation structure — a short AGENTS.md plus .agents/project_docs/ holding architecture.md, state.md, adr/ and an optional glossary, with nested per-area agent files where the repo has areas. Kept outside version control by default, with no archive file. Use when a project has no documentation structure, when its docs have grown into one unreadable file, when asked to organise or restructure project docs, or when asked to set up ADRs.
---

# Initialising a project's documentation

One-time and structural. For keeping it current — after work ships and at the
end of every session — that is `update-project-docs`, a separate skill, because
it runs constantly and this one runs once.

## The layout

```
AGENTS.md                  # short: commands, conventions, traps, routing table
.claude/CLAUDE.md          # one line, `@../AGENTS.md` — only if Claude Code is used
.agents/
└── project_docs/
    ├── architecture.md    # the mechanism, the invariants, the rules left behind
    ├── state.md           # what is decided but not built
    ├── glossary.md        # only when Step 1 says so
    └── adr/
        ├── README.md      # the index
        └── NNNN-slug.md
<area>/AGENTS.md           # only where an area has conventions of its own
```

Three fixed points, none of them a matter of taste — and one default: **the
docs are written in English** whatever language the conversation is in, unless
the project's `AGENTS.md` says otherwise.

- **`AGENTS.md` sits at the repository root and does not go into
  `project_docs/`.** It is the one file coding agents load by themselves, so it
  is paid for in every session — which is why the rest can be long and this one
  cannot. It carries the commands, the conventions, the traps, and a routing
  table pointing into `.agents/project_docs/`. Claude Code reads `CLAUDE.md`
  rather than `AGENTS.md`, so where it is used, `.claude/CLAUDE.md` holds the
  single line `@../AGENTS.md` and nothing else; the same one-line `CLAUDE.md`
  (`@AGENTS.md`) goes beside each `<area>/AGENTS.md`. Content written into a
  `CLAUDE.md` is content the other agents never see.
- **`project_docs/` is outside version control by default.** Add it to
  `.gitignore` unless the user asks for it to be tracked. The consequence is
  stated everywhere it matters: a delete is a destroy.
- **There is no archive file.** Text that stops being true is deleted. What
  deserves to outlive its truth is the *reason* for a choice, and that already
  survives as a superseded ADR, which is never deleted. A catch-all archive
  grows faster than everything else combined and is read almost never; it was
  measured at 578 KB against 118 KB for the architecture doc on the project
  this layout came from, with most of its reads being checks that nothing
  linked to it.

## The idea this is built on

Documentation rots **unevenly**. A command rots when the tooling changes, an
architectural rule rots when the architecture changes, a roadmap rots in days.
Put them in one file and the whole file becomes untrustworthy at the rate of its
fastest-rotting paragraph — and the usual symptom is a document that has to warn
you about itself: *"superseded, read the other section first."*

So the split is **by question asked and by staleness rate**, never by topic:

| the question | the file | rots when |
|---|---|---|
| how do I work here? | `AGENTS.md` | tooling changes |
| how does it work, and what must not break? | `project_docs/architecture.md` | the architecture changes |
| why this and not that? | `project_docs/adr/` | **never** — superseded, not edited |
| what is decided but not built? | `project_docs/state.md` | weekly |

And three rules that keep it from collapsing back into one file:

1. **`architecture.md` states the mechanism; an ADR states the choice.** "Reads
   are served from a cache keyed on the table version" is architecture. "We
   rejected a read replica because it puts a second copy of money in the system"
   is an ADR. The same sentence is never in both.
2. **An ADR is never edited when it stops being true, and never deleted.** It
   gets `Status: superseded by NNNN` and a *new* record is written. This is what
   makes the directory free of reading order — there is no chain to
   reconstruct — and, with no archive, it is where old reasoning lives.
3. **Reference material with a machine-readable source of truth is never copied
   into markdown.** The token file is the token reference, the schema DDL is the
   schema reference, the message catalogue is the string reference. A markdown
   copy is a second source that drifts, and drifts silently.

---

## Step 0 — detect, before writing anything

**Never scaffold blind.** Three things change the answer, and all three are
observable.

### Is this a scaffold or a migration?

```bash
ls README* CLAUDE.md AGENTS.md ARCHITECTURE.md docs/ .claude/ .agents/ 2>/dev/null
find . -maxdepth 3 -name '*.md' -not -path '*/node_modules/*' -not -path '*/.git/*' \
  | xargs wc -l 2>/dev/null | sort -rn | head -20
```

- **Nothing, or a thin README** → scaffold (Step 2A).
- **An accumulated doc, especially one over ~1,500 lines**, or docs in an older
  layout (a long `CLAUDE.md` or `AGENTS.md`, an `ARCHITECTURE.md` beside it, an
  `ARCHIVE.md`, a `docs/adr/`, docs under `.claude/`) → migration (Step 2B). This is the common case and the harder
  job.

**If the project carries its own docs skill** — a project-level `skills/*docs*` (under `.claude/`, `.agents/` or the like) whose
description says it is that project's copy — the project has deliberately kept
its own layout. Stop and say so; do not migrate it unless the user asks by name.

### What does git currently see?

```bash
git rev-parse --is-inside-work-tree 2>/dev/null && {
  git check-ignore -v AGENTS.md || echo "AGENTS.md WOULD BE TRACKED"
  mkdir -p .agents/project_docs && touch .agents/project_docs/.probe
  git check-ignore -v .agents/project_docs/.probe || echo "PROJECT_DOCS WOULD BE TRACKED"
  rm .agents/project_docs/.probe; }
```

If `project_docs/` would be tracked, add `.agents/project_docs/` to
`.gitignore` — unless the user asked for the docs to be in git, in which case
leave it tracked and say so. Leave `AGENTS.md`'s status as the project already
has it. Check rather than assume: a broad `.agents` or `.claude` ignore line,
or a tracked one, is common in both directions.

### Does the repo have distinct areas with their own conventions?

```bash
ls -d */ | head -20          # a monorepo, or frontend/ + backend/, or services/*
```

If yes, the answer is **nested agent files** (`frontend/AGENTS.md`,
`services/api/AGENTS.md`), not sections in the root one. They load only when
work happens in that directory, so a backend session pays nothing for frontend
conventions. Look specifically for conventions asserted in code and written down
nowhere — the tell is comments claiming a rule:

```bash
grep -rniE 'house rule|the rule is|deliberately|never |must not ' <area>/src --include='*.*' -l | wc -l
```

A high count against zero mentions in any doc is a real gap, and it is the one
most often missed: the design system, the component patterns, the test
conventions.

---

## Step 1 — settle the set

**Always:** `AGENTS.md`, `project_docs/architecture.md`,
`project_docs/adr/`, `project_docs/state.md`.

**Conditionally, and ask rather than guess:**

| file | create it when |
|---|---|
| `README.md` (repo root) | it is missing **and** a human other than the author will ever open the repo. It answers "what is this, how do I run it" for a person; `AGENTS.md` answers the agent's question. Not the same document, and a `README.md` is not a substitute for either |
| `project_docs/glossary.md` | a migration finds terms already defined somewhere, or the code plainly uses two words for one thing. Otherwise **do not create it here** — `lets-cook-together` creates it the first time a design session resolves a project-specific term, which is when there is something true to write. Format in Step 2A |
| `<area>/AGENTS.md` | Step 0 found an area with its own conventions |

**Do not create:** an archive file of any name; `CHANGELOG.md` unless the thing
is released and versioned; `CONTRIBUTING.md` unless outside contributions are
actually taken; a `PRODUCT.md` for content that is fifty lines — that belongs as
`architecture.md`'s first section, where the rest of the file reads as its
consequence. **A doc nobody has a reason to open is worse than a missing one:**
it renders as authoritative while nothing keeps it true.

---

## Step 2A — scaffold (greenfield)

Write each file with a header that states **what it refuses to carry**. That
sentence is the whole mechanism: without it, the next person appends the wrong
kind of content and the split is gone in a month.

Do not invent content. Read the code, write what is true, and leave a section
empty over guessing at it. An `architecture.md` full of plausible-sounding
architecture is worse than a short honest one.

Minimum viable versions:

- **`AGENTS.md`** — the commands that actually work (run them), the
  conventions, a **Traps** section (empty at first; it fills fast), and the
  routing table from Step 3. Short is the requirement, not the aspiration: it
  is loaded into every session.
- **`architecture.md`** — what the system is for, its parts and how they relate,
  the invariants, and an empty **The rules the work left behind** section.
  Carries **no dates**.
- **`adr/README.md`** — the index, the three-part bar, and the
  never-edit-never-delete-always-supersede rule.
- **`state.md`** — what is decided but unbuilt, with the **effect** of each gap
  rather than its title. Carries no dates.
- **`glossary.md`**, only when Step 1 called for it — one entry per term, and
  nothing but terms specific to this project; general programming concepts do
  not belong even where the code leans on them:

  ```md
  **Tenant**:
  A customer of the product; owns its accounts and its people.
  _Avoid_: customer account, org
  ```

  One or two sentences, what the thing *is* rather than what it does, and the
  words not to use for it. Group under subheadings once clusters appear. No
  implementation details: it is a glossary, not a spec.

---

## Step 2B — migrate (the usual case)

The order matters, because every step after the first depends on nothing having
been destroyed — and there is no archive to fall back on.

**1. Back up first, outside the repository, and verify the backup.** If the docs
are untracked this is the only safety net that exists, and the migration
deletes things on purpose. Copy them somewhere the migration will not touch
(`~/.agents/doc-backups/<project>-<date>/`), then `diff -r` — a backup nobody
checked is not a backup. Tell the user where it is.

**2. Measure before deciding.** The numbers decide the split, and they are
usually surprising:

```bash
wc -l -w *.md docs/*.md .claude/*.md .agents/*.md       # weight
grep -cE '\b20[0-9]{2}-[0-9]{2}-[0-9]{2}\b' <bigfile>    # is it a work log?
grep -coiE 'supersed|no longer (true|holds)|was wrong'   <bigfile>  # self-warnings
grep -c '^### ' <bigfile>                                # section count
```

A file where dated sections outweigh timeless content is a **decision journal
wearing a design doc's name**, and that diagnosis is what makes the rest
mechanical.

**3. Create `project_docs/` and move what already has the right shape.** An
existing `ARCHITECTURE.md`, `STATE.md` or `adr/` moves in under the lowercase
names, verbatim, and is diffed against the source. Never rewrite while moving.

**4. Extract the ADRs.** Test every candidate against all three:

1. **Hard to reverse.** If it is easy to reverse, you will just reverse it.
2. **Surprising without context.** A reader looks at the code and asks "why on
   earth did they do it this way?"
3. **The result of a real trade-off.** There were genuine alternatives.

Expect roughly a **quarter to a third** of dated sections to pass. Most of what
looks like a decision is a *finding*: "one answer was simply wrong" has no
alternative and is not an ADR. A **constraint imposed from outside** counts even
without alternatives, if it is invisible in the code — a third-party API that
answers a missing month with a zero instead of an error is exactly that, and it
is the kind of thing that produces a wrong screen nobody can diagnose.

Each ADR must **name the alternative** and **carry the measurement** if there
was one. An ADR without a rejected option is a description, and somebody will
propose the rejected thing again in six months. A rule without its measurement
reads as boilerplate and gets reasoned around.

**5. Extract what the findings leave behind, and let the story go.** What
survives goes to exactly one place:

| the finding leaves | where |
|---|---|
| a rule about how the system behaves or what it may claim | `architecture.md`, in **The rules the work left behind** |
| something that bites *while working* and is not guessable from the code | `AGENTS.md` → **Traps** |
| a constraint from outside the codebase | a new ADR after all |

State it as a rule, not a story: *"a folded top-N answer is not a lookup"*, not
*"on the 8th we found that…"*. Keep the figure that makes it land. The narrative
around it is not carried into the new set.

**An existing archive file is handled the same way**: read it for anything that
is a reason for a live choice with no ADR yet, write that ADR, and leave the
rest in the backup. Do not move an archive into `project_docs/`.

**6. Deduplicate by choosing, never by rewriting.** Where two files state the
same rule, keep **one existing text** and drop the other. Do not merge them
into new prose — you will lose a measurement, and you will not notice.

Check for this specifically; it is the most common finding and the most
valuable. **Where the same rule is stated twice, one copy is usually already
stale.** That is not a hypothetical cost of splitting, it is the realised one.

**7. Fix the links, and the positional ones are the trap.** Cross-file links and
anchors break loudly — and every one of them moves when the files move into
`project_docs/`. What breaks *silently* is every `see X below` and `see X above`
— the moment a section moves to another file, the direction is a lie, and a
false navigation instruction is worse than none.

```bash
grep -cE '\*\*[A-Z][^*]{4,70}\*\*( below| above)' AGENTS.md .agents/project_docs/*.md
```

Within-file `below`/`above` are fine and should be left alone.

**8. Delete the merged sources** only after diffing their content against where
it went, and name every deleted file in the report to the user.

---

## Step 3 — the routing table

Put it in `AGENTS.md`, and make every row a **trigger**, not a
description. This is what makes the split pay for itself:

> | when | read |
> |---|---|
> | you are working in this repo | `AGENTS.md` (this) |
> | how does it work, why is it shaped this way | `project_docs/architecture.md` |
> | why did we choose this over that, or why is it *not* X | `project_docs/adr/` → `README.md` |
> | what is decided but not built | `project_docs/state.md` |
> | what does this word mean here | `project_docs/glossary.md` (if it exists) |

A table saying what each file *is* does not work: the reader loads the biggest
file to answer a question the smallest one owns. Add the task-scoped triggers
too — *"touching the ingest path? this section is not optional reading."*

Then state, in the same section:

- the three rules from the top of this skill;
- that `project_docs/` is outside version control (or that it is tracked, if the
  user chose that), so a delete is a destroy;
- **that the docs are updated by `/update-project-docs` when a piece of work
  ships and at the end of every session.** A convention with no moment attached
  is the one that decayed in the first place.

---

## Step 4 — verify, do not reason

Run from the repository root.

```bash
# every link resolves, relative to the file it is written in
for f in AGENTS.md .agents/project_docs/*.md .agents/project_docs/adr/*.md; do
  d=$(dirname "$f")
  grep -oE '\]\([^)#]*\.md' "$f" | sed 's/^](//' | sort -u | while read l; do
    [ -f "$d/$l" ] || echo "BROKEN in $f: $l"; done
done

# the dateless files are dateless
grep -cE '\b20[0-9]{2}-[0-9]{2}-[0-9]{2}\b' .agents/project_docs/architecture.md .agents/project_docs/state.md

# git sees what it was meant to
git check-ignore -v .agents/project_docs/architecture.md

# the weight
wc -l AGENTS.md .agents/project_docs/*.md
```

On a migration, also confirm **every section heading from the old files is
either somewhere in the new set or was deliberately dropped** — list the
dropped ones for the user. That check catches an off-by-one range extraction,
which is the one mistake that silently drops content.

A date-shaped value can be legitimate — a start date passed to an API is data.
A `Shipped 2026-…` opener is not, and in a dateless file it means the ritual was
skipped.

---

## What this skill will not do

- **Write documentation from imagination.** Every claim comes from the code, a
  command that was run, or the user. An architecture doc that describes what the
  system probably does is a liability.
- **Delete anything without saying so.** The docs are untracked by default, so a
  delete is a destroy; the backup in Step 2B and the list in the report are what
  make it a decision rather than an accident.
- **Create a file for content that does not exist yet.** Empty scaffolding
  invites the wrong content.
- **Create an archive file**, under any name.
- **Overwrite an existing `README.md`.** It is usually the one document with a
  human audience.
