---
name: test-the-meal
description: "Write real tests for the work just built — two sub-agents in parallel, one testing the agreed behaviour at its edges, one trying to break it — never the happy path alone, each test proven able to fail, then the full suite once. Reads what changed since the last commit (or a ref or path you give it). Does one job: tests. `trim <path>` instead slims an overgrown test file, and only when asked."
disable-model-invocation: true
---

# Test the meal

`/implement` builds without writing tests. This skill writes them, for the work
that was just built, and does nothing else: no project docs, no commit.

**Language.** Talk in the user's language. Tests and test names are in
**English**, unless the project's `AGENTS.md` says otherwise.

**Modes**, from what the user typed after the command:

| typed | does |
|---|---|
| nothing | tests for what changed since the last commit, uncommitted work included |
| a git ref or range (`a1b2c3d`, `main~8..`) | tests for what changed since that ref or in that range |
| a path (`backend/alerts/`) | tests for that area only |
| `trim <path>` | **no new tests.** Slims one test file or directory (step 7). Only when the user types it |

---

## 1. What to test

Collect the changed files for the mode (`git diff --stat HEAD` plus untracked
files, by default). Drop what a test cannot usefully reach — styling,
configuration, migrations, generated code, build scripts — and name it in the
report.

From the changed code, list the **seams**: the public boundaries where the new
behaviour is observed without reaching inside — an HTTP endpoint, a public
function or class, a rendered component as the user sees it, a CLI command.
Tests go **only at seams**.

**The agreed behaviour** is the `lets-cook-together` summary and the
`/implement` report if they are in this conversation; otherwise the ADRs and
docs the change touches. Expected values come from there, not from the code.

Read `AGENTS.md` and the per-area one for the test commands, fixtures and
traps, and find **the existing tests of each seam** before anything is written.

**Pin the starting point:** `git rev-parse HEAD`, and say in one line if the
tree is dirty (it usually is — that is the work being tested).

## 2. Two agents, in parallel

Start two sub-agents in the same turn. Give each the seams, the changed files,
the agreed behaviour (the summary, quoted), the existing test files of those
seams, the test command, and the rules in step 3. **They write nothing to the
repository.** Each returns its tests as code, each with its target file and the
one small code edit that should make it fail. Two agents editing the same files
— or breaking the same code to prove a test — would collide; the main session
applies and proves them one at a time (step 4).

- **Agent A — the agreed behaviour, at its edges.** Every rule the summary
  states, tested where it is most likely to be wrong: boundaries and limits,
  empty and missing values, the second and the many (not only the one),
  ordering, state transitions, the error paths the agreement names, and both
  languages or time zones where the product has them.
- **Agent B — trying to break it.** What the agreement did not think of:
  permissions and the wrong user or tenant, invalid and hostile input, a
  dependency failing or answering slowly, a repeated or concurrent action, and
  the neighbouring behaviour this change could have broken.

**Neither writes a happy-path-only test.** A test whose input is the ordinary
case and whose assertion is "it worked" belongs only where the existing suite
has no coverage of the seam at all, and then as one case, never the whole set.

**Budget, per changed seam: at most three tests per agent.** Prefer a new case
in an existing test (a row in a parametrized table, one more scenario in a file
that already sets up the same state), then a new test in an existing file, and
a new file last. One test per behaviour, not per function.

## 3. What a test must be

**Real.** It verifies behaviour through the public interface and survives a
rewrite of the internals. Its name says *what* the capability is ("a member
cannot open another tenant's account"), not *how* it is done.

**Independent of the code it checks.** Expected values come from the agreement,
a worked example or a known literal — never recomputed the way the code
computes them, never pasted from running the code. **When the code and the
agreement disagree, the test follows the agreement** and fails: that is a
finding, not a test to bend.

**Not:**

- **implementation-coupled** — mocking internal collaborators, testing private
  functions, asserting call counts or order, verifying through a side channel
  (querying the database instead of reading back through the interface);
- **tautological** — passing by construction;
- **a snapshot of everything** — a large snapshot or full-object equality where
  two fields matter.

**Mock at system boundaries only** — external APIs, time, randomness, sometimes
the database or the file system. Never the project's own modules. Reuse the
existing fixtures and factories.

## 4. Apply and prove

When both agents return, take their tests one file at a time:

1. Write the tests into their target file, merging where both agents chose the
   same file. Drop a test that duplicates another one — from the other agent or
   from the existing suite.
2. Run that file. A test that fails because it is written wrong is fixed; a test
   that fails because **the code disagrees with the agreement** is a finding
   (step 5).
3. **Prove each new test can fail.** Make its one small code edit (flip a
   condition, return early), run it, watch it fail, undo the edit. Confirm with
   `git diff` that no mutation is left. A test that still passes with the
   behaviour broken is rewritten or deleted.

## 5. Findings

A test that fails against the agreed behaviour is reported, not hidden: the
test, the agreed line, and what the code does instead. Ask **one short
question** for all of them together — fix the code now, or leave it for later.
On "fix now", fix the code, not the test, and re-run.

## 6. Run it

- **The full suite once, at the end**, and the typecheck. `/implement` runs only
  the area's tests, so this is where cross-area regressions surface. A failure
  outside this run's area is reported, not fixed here.

Use the commands `AGENTS.md` gives, and heed its traps.

## 7. `trim <path>` — only when the user types it

No new tests. In that path, find the tests that only add weight: duplicates of
one behaviour at one seam, near-identical tests that should be one parametrized
test, implementation-coupled tests whose behaviour is covered at a seam, tests
of behaviour that no longer exists, and the same long setup built inline many
times. **Propose, do not apply:** each test, the action, **which remaining test
covers the behaviour afterwards**, and the rough lines saved. Apply only what
the user approves, then run those files again. Never weaken an assertion to
merge two tests, and never delete the only coverage of a behaviour the docs
name.

## 8. Report

End with a short report in the user's language:

- per seam: the tests added, by agent, and where (new case, new test, new file);
- tests dropped as duplicates or because they could not fail;
- files left without tests, and why;
- findings, and what was decided about each;
- the full suite and the typecheck as run, with the failing output if something
  failed.

Then stop. The docs and the commit are `/update-project-docs`.
