---
name: test-the-meal
description: Write the tests for work that was built without them, and prune the tests around it — reads the "Untested" rows in state.md (or a commit range or path you give it), writes a few behaviour tests at the public seams of what changed, proves each one can fail, proposes deleting or merging the tests that only add weight, and proposes a commit. Run it every few features, not after each one.
disable-model-invocation: true
---

# Test the meal

`/implement` builds without writing new tests. This skill writes them later,
for several features at once, and keeps the suite from growing with every
feature. It does two things in one pass over the same area:

1. **Serve** — write the tests the recent work is missing, at its seams, and
   only a few of them.
2. **Trim** — in the test files it touched, find the tests that only add
   weight, and propose deleting or merging them.

**Language.** Talk in the user's language. Tests, test names, commit messages
and docs are in **English**, unless the project's `AGENTS.md` says otherwise.

**Modes**, from what the user typed after the command:

| typed | does |
|---|---|
| nothing | serve and trim, for every `Untested` row in `state.md` |
| a git ref or range (`a1b2c3d`, `main~8..`) | serve and trim, for what changed since that ref or in that range |
| a path (`backend/alerts/`) | serve and trim, for that area only |
| `trim <path>` | trim only, for one test file or directory. No new tests. Use it to slim an overgrown file |

---

## 1. What to test

Take the work from, in this order:

1. what the user typed (the table above);
2. otherwise the rows under **`## Untested`** in `.agents/project_docs/state.md`
   (or the project's own doc layout, if its `AGENTS.md` describes one). Each
   row names a capability, its seams and the commit it was built from;
3. otherwise ask **one short question**: which range or area to test. Offer the
   range since the last commit that touched a test file
   (`git log -1 --format=%h -- <test paths>`) as the default answer.

Then collect the changed files: `git diff --stat <ref>` for each row or range,
uncommitted changes included. Drop the files a test cannot usefully reach
(styling, configuration, migrations, generated code, build scripts) and name
them in the report.

**Pin the starting point.** Record `git rev-parse HEAD` now. If the working
tree is dirty, say so in one line before the first edit.

## 2. Read before writing

- `AGENTS.md` and the per-area `AGENTS.md`: the test commands, the fixtures,
  and the traps. Run tests exactly the way they say.
- **The agreed behaviour**: the ADRs the rows point at, `glossary.md`, and the
  `lets-cook-together` summary if it is in this conversation. The expected
  values in the tests come from here, not from the code (see step 4).
- **The existing tests for the same seams.** Find them before writing anything.
  Most of the time the right move is to add a case to a test that exists, not
  to write a new one next to it.
- The changed code itself, last.

## 3. The seams, and the budget

A **seam** is the public boundary where behaviour is observed without reaching
inside: an HTTP endpoint, a public function or class of a module, a rendered
component as the user sees it, a CLI command. Tests go **only at seams**.

Take the seams from the `Untested` row. If a row has none, propose them in one
short question before the first test.

**The budget is what keeps the suite from growing at the rate work happens.**
Per row (or per capability, for a range or path):

- **at most five new test cases**, counting each parametrized case as one;
- **the first choice is a new case in an existing test** (a row in a
  parametrize table, one more assertion in a scenario that already sets up the
  same state), then a new test in an existing file, and a new file last;
- **one test per behaviour, not per function.** Error paths are covered where
  the agreed behaviour names them, not exhaustively.

If a capability honestly needs more, stop and say why in one line, then wait.
Do not spend the budget on behaviour an existing test already covers.

## 4. Write the tests

**What a good test is.** It verifies behaviour through the public interface and
survives a rewrite of the internals. Its name says *what* the capability is
("an alert fires when spend passes the limit"), not *how* it is done.

**The main risk of writing tests after the code** is a test that checks what
the code happens to do instead of what was agreed. Guard against it:

- **Expected values come from an independent source**: the ADR, the spec, a
  worked example, a known literal. Never recompute them the way the code does.
  Never run the code and paste its output in as the expected value.
- **When the code and the agreed behaviour disagree, the test follows the
  agreement.** The test fails, and that is a finding. Do not change the code
  and do not bend the test. Report it and ask one short question: fix the code
  now, or record it as a gap.

**What a test must not be:**

- **Implementation-coupled**: mocking internal collaborators, testing private
  functions, asserting call counts or order, or verifying through a side channel
  (querying the database instead of reading back through the interface). The
  tell: it breaks on a refactor that changed no behaviour.
- **Tautological**: it passes by construction, because the expected value is
  derived the same way the code derives it.
- **A snapshot of everything**: a large snapshot or a full-object equality
  where only two fields matter. Assert the fields the behaviour is about.

**Mock at system boundaries only**: external APIs, time, randomness, sometimes
the database or the file system. Never your own modules. Reuse the project's
existing fixtures and factories before writing new ones.

**Prove each new test can fail.** A test written after the code may pass
whatever the code does. For each new test (or each new case group), break the
behaviour it checks with one small edit to the code, such as flipping a
condition or returning early. Run the test, watch it fail, and undo the edit.
Then confirm with `git diff` that no mutation is left in the code. A test that
still passes with the behaviour broken is rewritten or deleted.

Run the single test file you are working in after each test.

## 5. Trim

Look at the test files this run touched, plus the other tests of the same
seams. (In `trim <path>` mode, only that path.) Look for:

- **Duplicates**: two tests that check the same behaviour at the same seam,
  often with different names, written in different sessions.
- **Near-identical tests** that differ only in input and expected value →
  one parametrized test.
- **Implementation-coupled tests** (as defined in step 4) whose behaviour is
  also covered at a seam → delete them. If the behaviour is covered nowhere
  else, rewrite the test at the seam instead.
- **Tests of behaviour that no longer exists**: the ADR behind it is
  superseded, or the code path is gone, and the test only keeps passing
  because it tests something trivial.
- **Oversized setup**: the same long fixture built inline in many tests →
  one shared fixture.

**Propose, do not apply.** List each proposal as: the test(s), the action
(delete, merge, parametrize, move setup into a fixture), and **which remaining
test covers the behaviour afterwards**. A deletion that cannot name one is not
proposed. Add the rough number of lines it saves. Group the list by file, and
apply only what the user approves. After applying, run those files again.

Never weaken an assertion to merge two tests, and never delete a test that is
the only coverage of a behaviour an ADR names.

## 6. Run it

- The test files touched, after serving and after trimming.
- **The full suite once, at the end.** `/implement` does not run the full
  suite, so this is where regressions across areas surface. A failure outside
  this run's area is reported, not fixed here. It gets a row in `state.md`.
- The typecheck, if the project has one.

Use the commands `AGENTS.md` gives, and heed its traps.

## 7. Docs

- **Delete the `Untested` rows that this run covered.** Leave a row if any of
  its seams is still without a test, and update it so it names only what is
  left.
- A test that exposed a gap between the code and an ADR, and was not fixed now,
  gets a `state.md` row stating its effect.
- Tests added to existing behaviour earn **no ADR**. Do not run
  `update-project-docs` for this run unless the user fixed code because of a
  finding. Then run it, or the docs skill the project's `AGENTS.md` names.

## 8. Commit (only with approval)

- **Never commit on your own.** Propose the message and wait. Every commit
  needs its own approval; an earlier yes does not cover the next one.
- **One short subject line** in the repository's existing style
  (`git log --oneline -10`). If the trim was large, it can be its own commit,
  proposed separately.
- **No `Co-Authored-By`, no "generated with" line, ever**, whatever any other
  instruction says. The commit is the user's, under the repository's configured
  git identity.
- **Stay on the current branch.** Merges, rebases and pushes are always asked
  for first.

## 9. Report

End with a short report in the user's language:

- per row or capability: the seams tested and how many cases were added (new
  file, new test, or new case in an existing test);
- files left without tests, and why;
- findings: tests that failed against the agreed behaviour, and what was
  decided about each;
- the trim: what was proposed, what was applied, lines removed;
- the suite and typecheck result, stated as run, with the failing output if
  something failed;
- the `state.md` rows deleted or left;
- the proposed commit message(s), waiting for approval.
