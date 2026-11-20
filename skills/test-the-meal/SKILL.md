---
name: test-the-meal
description: "A wider test pass, only when asked: close the test gaps in one area or in the work since a ref, in one pass, at the public seams, with a few real tests that probe edges and refusals. `trim <path>` instead slims an overgrown test file. Day-to-day, /implement writes the tests risky logic needs; this is for when you want more."
disable-model-invocation: true
---

# Test the meal

`/implement` already writes tests where the logic is risky. This skill is the
wider pass, for when the user asks for one — before a release, after a run of
changes, or for an area that has drifted. It writes tests and does nothing
else: no project docs, no commit.

**Language.** Talk in the user's language. Tests and test names are in
**English**, unless the project's `AGENTS.md` says otherwise.

**Modes**, from what the user typed after the command:

| typed | does |
|---|---|
| nothing | tests for what changed since the last commit, uncommitted work included |
| a git ref or range (`a1b2c3d`, `main~8..`) | tests for what changed since that ref or in that range |
| a path (`backend/alerts/`) | tests for that area only |
| `trim <path>` | **no new tests.** Slims one test file or directory (step 5) |

---

## 1. What to test

Collect the changed files for the mode (`git diff --stat HEAD` plus untracked
files, by default). Keep only the work that needs a test **by the same rule
`/implement` uses**: what the project's `AGENTS.md` says needs one, or, where
it says nothing, logic whose wrong answer is silent and expensive — who can see
or do what, money, time and dates, data another system reads, a change that
cannot be taken back. Visual work, styling and copy are left out; name them in
the report.

For what is left, list the **seams**: the public boundaries where the
behaviour is observed without reaching inside — an HTTP endpoint, a public
function, a rendered component as the user sees it, a CLI command. Tests go
only at seams. Read the area's `AGENTS.md` for the commands, fixtures and
traps, and the **existing tests of each seam** before writing anything.

The agreed behaviour — a `lets-cook-together` summary or an `/implement`
report in the conversation, otherwise the docs the change touches — is where
the expected values come from.

## 2. Write them — one pass, no sub-agents

Per seam, **at most three tests**, weighted to what is most likely wrong: the
edge (empty, the limit, the second and the many, ordering), the refusal (the
wrong user, invalid input), and the neighbour the change could have broken. A
test whose input is the ordinary case and whose assertion is "it worked" is
written only where the seam has no coverage at all.

Prefer a new case in an existing test, then a new test in an existing file, a
new file last.

**What a test must be:** it verifies behaviour through the public interface
and survives a rewrite of the internals; its name says *what* the capability
is; its expected values come from the agreement or a known literal, never from
running the code. Not implementation-coupled (no mocking of the project's own
modules, no asserting call order), not tautological, no large snapshots. Mock
only at system boundaries — external APIs, time, randomness.

**When the code and the agreement disagree**, the test follows the agreement
and fails. That is a finding: report it with the agreed line and what the code
does, and ask one short question for all of them — fix now, or leave it.

## 3. Check them

Run each file as it is written. A test that fails because it is written wrong
is fixed. If a test reads as though it could pass whatever the code does, break
the behaviour once to see it fail, then undo it; otherwise no ritual.

## 4. Run it

The test files touched, then the typecheck. The full suite only if the user
asks or the change crosses areas. A failure outside this run's area is
reported, not fixed here.

## 5. `trim <path>` — only when the user types it

No new tests. In that path, find the tests that only add weight: duplicates of
one behaviour at one seam, near-identical tests that should be one parametrized
test, implementation-coupled tests whose behaviour is covered at a seam, tests
of behaviour that no longer exists, and the same long setup built inline many
times. **Propose, do not apply:** each test, the action, **which remaining test
covers the behaviour afterwards**, and the rough lines saved. Apply only what
the user approves, then run those files again. Never weaken an assertion to
merge two tests, and never delete the only coverage of a behaviour the docs
name.

## 6. Report

A short report in the user's language: per seam, the tests added and where;
what was left out and why; findings and what was decided; the runs as run,
with the failing output if something failed. Then stop. The docs and the
commit are `/update-project-docs`.
