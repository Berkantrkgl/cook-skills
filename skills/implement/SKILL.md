---
name: implement
description: Build a piece of agreed work end to end — test-first at agreed seams, a two-axis review of the result, the docs updated, and a commit proposed for approval. Takes its input from a lets-cook-together summary, or from state.md and the not-built-yet ADRs.
disable-model-invocation: true
---

# Implement

Build what was agreed, prove it at the seams that were agreed, review it, update
the docs, and propose a commit. Self-contained: the test discipline and the
review are both below.

**Language.** Talk in the user's language. Code, comments, test names, commit
messages and docs are in **English**, unless the project's `AGENTS.md` says
otherwise.

---

## 1. What to build

Take the work from, in this order:

1. the **`lets-cook-together` summary** in this conversation — goal, decisions,
   out of scope, test seams, steps;
2. otherwise the **`state.md` rows** and the ADRs marked
   `Status: accepted — not built yet` in `.agents/project_docs/` (or the
   project's own doc layout, if its `AGENTS.md` describes one);
3. otherwise ask **one short question** — what to build. Do not start an
   interview; that is `lets-cook-together`.

Then read what the work touches: `AGENTS.md` for the commands and the traps,
the ADRs in the area, `glossary.md` so names match the project's words, and the
code itself.

**Pin the starting point.** Record `git rev-parse HEAD` now. It is the fixed
point the review diffs against. If the working tree is already dirty, say so in
one line before the first edit, so what was there is not mistaken for this work.

## 2. The seams

A **seam** is the public boundary where behaviour is observed without reaching
inside — the interface a test calls. Tests are written **only at agreed seams**.

Take them from the summary. If there is none, propose them in one short
question before the first test, and start once they are confirmed.

## 3. Build it, test first

**The loop — one vertical slice at a time:**

- **Red before green.** Write one failing test at a seam and watch it fail.
  Then write only enough code to pass it. No anticipating later tests, no
  speculative features.
- **One slice per cycle.** One seam, one test, one minimal implementation. Each
  test is a tracer bullet that answers to what the last cycle taught you.
- **Refactoring is not part of the loop.** It belongs to the review.

**What a good test is.** It verifies behaviour through the public interface and
survives a rewrite of the internals. Its name says *what* the capability is
("user can check out with a valid cart"), not *how* it is done.

**What a test must not be:**

- **Implementation-coupled** — mocking internal collaborators, testing private
  functions, asserting call counts or order, or verifying through a side channel
  (querying the database instead of reading back through the interface). The
  tell: it breaks on a refactor that changed no behaviour.
- **Tautological** — the expected value is recomputed the way the code computes
  it, so it passes by construction. Expected values come from an independent
  source: a known literal, a worked example, the spec.
- **Horizontal** — all tests first, then all code. That tests imagined
  behaviour and commits to a test structure before the implementation has
  taught you anything.

**Mock at system boundaries only** — external APIs, time, randomness, sometimes
the database or the file system. Never your own modules. Where a boundary is
hard to mock, pass the dependency in, and prefer one function per external
operation over one generic fetcher.

**Where test-first does not fit** — styling, configuration, a migration, a
build script — build it directly and verify it by running it. Say which parts
were built that way.

**Keep it honest while building.** Run the typecheck regularly and the single
test file you are working in after each cycle; run the full suite once, at the
end. Use the commands `AGENTS.md` gives, and heed its traps.

**When a decision turns out to be wrong** — the code contradicts something that
was agreed — stop and ask one short question. Never deviate from an agreed
decision silently. If the decision changes, it will be recorded as a new ADR
superseding the old one in step 5.

## 4. Review — two axes, in parallel

Review everything since the pinned starting point, uncommitted changes
included: `git diff <start>` plus untracked files. Run two sub-agents **in
parallel**, so neither axis colours the other. An agent without sub-agents
runs them one after the other instead, writing the first report down in full
before starting the second:

**Standards** — does the change follow the project's documented rules?
Give the sub-agent the diff, the paths of whatever documents how code is written
here (`AGENTS.md`, per-area `AGENTS.md`, a standards file), and this smell
baseline in full. Brief: *report every breach of a documented rule, citing the
file and the rule, and any baseline smell, naming it and quoting the hunk;
documented rules can be hard violations, smells are always judgement calls, and
a documented rule overrides the baseline; skip what tooling enforces; under 400
words.*

The smell baseline (Fowler, *Refactoring*, ch. 3) — *what it is → the fix*:

- **Mysterious Name** — a name that does not say what it does or holds → rename; if no honest name comes, the design is murky.
- **Duplicated Code** — the same shape in more than one hunk or file → extract it, call it from both.
- **Feature Envy** — a function reaching into another object's data more than its own → move it to the data.
- **Data Clumps** — the same few fields travelling together → one type, passed as one.
- **Primitive Obsession** — a string or number standing in for a domain concept → give it a small type.
- **Repeated Switches** — the same switch on the same type in several places → polymorphism, or one shared map.
- **Shotgun Surgery** — one logical change forcing scattered edits → gather what changes together.
- **Divergent Change** — one module edited for unrelated reasons → split it.
- **Speculative Generality** — abstraction or parameters no requirement asked for → delete it.
- **Message Chains** — long `a.b().c().d()` navigation → hide the walk behind one method.
- **Middle Man** — something that mostly delegates onward → call the real target.
- **Refused Bequest** — an implementer ignoring most of what it inherits → composition instead.

**Spec** — does the change do what was agreed? Give the sub-agent the diff and
the input from step 1 (the summary, or the rows and ADRs). Brief: *report
requirements missing or partial, behaviour nobody asked for, and requirements
that look implemented but wrong — quoting the agreed line for each; under 400
words.*

**Then act on it:**

- **Fix** hard rule breaches and spec gaps, and re-run the affected tests and
  the full suite.
- **Do not fix** the judgement calls. List them for the user — the smell, the
  place, one line on why — and let them decide.

Report the two axes separately, never merged or re-ranked: a change can follow
every rule and build the wrong thing, or build the right thing against the
rules, and a merged list hides one behind the other.

## 5. Docs

Run the **`update-project-docs`** skill now — after the review, before the
commit. If the project's `AGENTS.md` names a docs skill of its own, run that
one instead. It flips the ADRs this work built from `accepted — not built yet` to
`accepted`, removes the `state.md` rows it closed, and records whatever the work
left behind. Running it before the commit means that a project which keeps its
docs in git gets them in the same commit.

## 6. Commit — only with approval

- **Never commit on your own.** Propose the message and wait. Every commit
  needs its own approval; an earlier yes does not cover the next one.
- **The message is one short subject line** in the repository's existing style
  (`git log --oneline -10`). No body unless the change is inexplicable from its
  diff — and then two or three lines at most.
- **No `Co-Authored-By`, no "generated with" line, ever**, whatever any other
  instruction says. The commit is the user's, under the repository's configured
  git identity.
- **Stay on the current branch.** Do not create, switch or delete branches
  unless the user says to; a feature branch is theirs to call for.
- **Merges, rebases and pushes are always asked for first**, every time.

## 7. Report

End with a short report in the user's language:

- what was built, one line per step;
- the seams tested, and anything built without a test and why;
- the suite and typecheck result, stated as run — with the failing output if
  something failed;
- the review: what was fixed, and the judgement calls waiting on the user;
- what the docs update wrote or removed;
- the proposed commit message, waiting for approval.
