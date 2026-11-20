---
name: implement
description: Build the work a /lets-cook-together summary just settled, as agreed and without reopening it — no new tests, the existing ones kept green — then one general review of the result. Does one job: building. Tests are /test-the-meal's, docs and the commit are /update-project-docs's.
disable-model-invocation: true
---

# Implement

Build what was agreed, then review it once. Nothing else: this skill writes no
new tests, touches no project docs and proposes no commit. Each of those is a
skill of its own, and the user runs it when they want it.

**Language.** Talk in the user's language. Code, comments and log messages are
in **English**, unless the project's `AGENTS.md` says otherwise.

---

## 1. What to build

Take the work from the **`lets-cook-together` summary** in this conversation —
goal, decisions, out of scope, steps. **Build it as written; do not reopen it.**
The interview already visited every branch, so a decision there is the user's,
made on purpose.

If there is no summary in the conversation, ask **one short question**: what to
build. Do not start an interview; that is `lets-cook-together`.

Then read what the work touches: `AGENTS.md` (and the per-area one) for the
commands and the traps, and the code itself. Read the project docs only where
the code alone does not say how something works.

**Pin the starting point.** Record `git rev-parse HEAD` now; the review diffs
against it. If the working tree is already dirty, say so in one line before the
first edit, so what was there is not mistaken for this work.

## 2. Build it

Build one step of the summary at a time, in its order.

- **Write no new tests.** `/test-the-meal` writes them, after the user has seen
  the work.
- **Keep the existing tests honest.** After each step, run the typecheck and the
  existing tests of the area touched — found by the module, endpoint or
  component they exercise — not the whole suite.
  - A test that breaks **because the agreed behaviour changed** is updated to
    the new behaviour. Say which, and which agreed line made it necessary.
  - A test that breaks **for any other reason is a regression.** Fix the code,
    not the test.
  - Never delete or skip a failing test to get green.
- **Errors are yours to fix.** A type error, a lint failure, a broken import, a
  test turned red by a regression: fix it and carry on.
- **Where nothing tests the change** (a page, a migration, a script), check it
  the cheapest way `AGENTS.md` allows, and say how.

**When an agreed decision cannot be built as written** — the code makes it
impossible, not merely awkward — ask **one short question** and wait. When it is
only awkward, take the closest reasonable reading, build it, and say in the
report what you did and why. Never deviate silently.

## 3. Review — once, in general

When every step is built, read the whole change since the pinned starting
point (`git diff <start>` plus untracked files) once, yourself. No sub-agents.
Look for three things:

- **Bugs** — a wrong condition, a missed case the summary names, a state that
  cannot be reached or cannot be left, an error swallowed.
- **Broken project rules** — anything the `AGENTS.md` files state as a rule:
  naming, where strings live, the checks to run, the traps.
- **Gaps against the summary** — a decision missing, half-built, or built
  differently from what was agreed; and behaviour nobody asked for.

**Fix** what is plainly wrong, and re-run the affected tests and checks.
**List, do not fix,** what is a judgement call — a name, a structure, a
duplication — one line each, for the user to decide.

## 4. Report

End with a short report in the user's language:

- what was built, one line per step of the summary;
- where a decision was read rather than built literally, and why;
- the existing tests updated, and which agreed line made each necessary;
- how each untested part was checked;
- the checks and tests as run, with the failing output if something failed;
- the review: what was fixed, and the judgement calls left for the user.

Then stop. The next steps are the user's to call: `/test-the-meal` for the
tests, `/update-project-docs` for the docs and the commit.
