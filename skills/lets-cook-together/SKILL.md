---
name: lets-cook-together
description: A relentless design interview that sharpens a plan until we share one understanding of it — round by round, a decision tree, the project's language made exact on the way — and then records what was decided (ADRs, glossary, state.md) and hands /implement a summary it can build from.
disable-model-invocation: true
---

# Let's cook together

Interview the user about a plan, a design or an idea until the two of you reach
a **shared understanding** of it. Then, and only then, write down what was
decided and hand it over to be built.

**Language.** Talk in whatever language the user is writing in, and keep to it
for the whole session. Everything written to a file is in **English**, unless
the project's `AGENTS.md` says otherwise.

---

## 1. Read before asking

Before the first round, read what the project already knows, so that no
question asks for something written down:

- `AGENTS.md` (and the routing table in it)
- `.agents/project_docs/`: `architecture.md`, `state.md`, `adr/README.md`,
  `glossary.md` if it exists — or, if the project describes its own doc layout
  in `AGENTS.md`, the equivalents it names
- the code the topic touches

Then say in two or three sentences what you understood the topic to be, and
start the first round.

## 2. The design tree

Map the topic as a **design tree**: every decision branches into the decisions
that hang off it.

The **frontier** is every decision whose prerequisites are already settled —
the questions that can be asked *now* without guessing at answers not yet
given. A question whose answer depends on another question still open belongs
to a later round.

When the topic is something to be built, the tree always has one more branch:
**where the behaviour will be tested** — the seams `/implement` will write its
tests against. Ask it once what is being built is settled; it is not a detail
to leave to implementation.

**Facts are yours, decisions are the user's.** When a question needs a fact
from the code, the files or the tools, dispatch a sub-agent to find it (or
look it up yourself, if the agent has no sub-agents); never ask the user for
anything you could look up. Do not block on it: a running
search is an unsettled prerequisite, so only the questions downstream of it
wait, and the rest of the frontier is asked now.

## 3. Rounds

**At most four questions per round.** If the frontier is larger, ask first the
ones that unblock the most decisions behind them, and keep the rest for the
next round.

Each question is **short and to the point**: the decision, the options if there
are any, and one recommendation. No background story, no measurements, no
history unless the user asks for them or the decision cannot be made without
one fact — and then that one fact, in a clause.

The format, exactly — note the empty line after every divider:

```
❓ **Q1** - **<title>**: <the question>

➡️ <your recommended answer>

---

❓ **Q2** - **<title>**: <the question>

➡️ <your recommended answer>
```

**Never use a question tool** (AskUserQuestion or similar), not even as a
clickable copy of a round already written. The cards make an answer too easy
to pick without reading, and cut the context a recommendation carries.

Then wait for the answers. Each answer reshapes the tree: settled decisions push
the frontier outward. Recompute it and ask the next round. An answer that is a
question back, or an objection, is not a settled decision — answer it and
re-ask.

## 4. Make the language exact, inside the rounds

While the tree is worked, the project's words are sharpened too. None of this
is a separate phase; each one becomes a question in the current round when it
comes up:

- **Against the glossary.** The user uses a term differently from how
  `glossary.md` defines it: *"the glossary says 'account' is X, you seem to mean
  Y — which is it?"*
- **Fuzzy words.** A vague or overloaded term gets a proposed canonical word:
  *"'account' — the tenant, or the cloud account it connects?"*
- **Scenarios.** Relationships between concepts are stress-tested with
  concrete, invented edge cases that force the boundary to be stated.
- **Against the code.** When the user says how something works, check whether
  the code agrees, and surface a contradiction as a question.

## 5. Nothing is written during the session

No file is touched while the rounds run. A decision reversed three rounds later
would otherwise leave a record that has to be superseded before it was ever
true. Keep the settled decisions and resolved terms in the conversation.

## 6. Ending

**The normal end:** the frontier is empty — every branch visited, nothing
silently assumed. Say so, and ask the user to confirm that you have reached a
shared understanding.

**The early end:** the user says to stop ("enough", "go ahead", "uygula" or the
like). Stop asking immediately. Show, in one list, the decisions still open and
what you are assuming for each, and ask **one** confirmation: *these are my
assumptions — agreed?* Do not reopen the rounds unless the user rejects one.

Nothing is recorded and nothing is built before that confirmation.

## 7. Record what was decided

After the confirmation, write everything in one pass.

**Where.** Into `.agents/project_docs/`. If the project has no `project_docs/`
but its `AGENTS.md` describes a doc layout of its own, write into that layout
instead, following its rules. If it has neither, ask one question here, not at
the start — *"there is no `project_docs/`; shall I run `/init-project-docs` and
record into it?"* On yes, run the `init-project-docs` skill, then record. On no,
write nothing; the summary in step 8 is then the only record, so say so. Never
run it at the start: on a project that already has docs, init is a migration
that deletes files, and it does not belong in the middle of a design talk.

**ADRs** — one per decision that passes all three:

1. **Hard to reverse** — changing your mind later costs something real.
2. **Surprising without context** — a future reader would ask "why on earth
   this way?"
3. **The result of a real trade-off** — there were genuine alternatives.

A constraint imposed from outside (a vendor limit, an API's behaviour, a
compliance rule) counts even without alternatives, if it is invisible in the
code. Most decisions of a session do **not** pass, and that is correct.

Each ADR: `adr/NNNN-short-slug.md`, one above the highest number, a title and a
few paragraphs, with **`Status: accepted — not built yet`** under the title. It
names the rejected alternative, carries any figure that decided it, and does
not describe the mechanism (that is `architecture.md`'s job, once it is built).
Add its row to `adr/README.md`. One decision per record, even when several were
made together.

**Glossary** — each project-specific term the session resolved goes into
`glossary.md`, created with the first one:

```md
**Tenant**:
A customer of the product; owns its accounts and its people.
_Avoid_: customer account, org
```

One or two sentences on what it *is*, the words not to use for it, no
implementation detail. General programming concepts do not belong, however much
the code uses them.

**state.md** — each piece of work the session decided on gets a row stating its
**effect**, not its title, so the plan survives the session ending.

`architecture.md` is **not** touched: it describes what is built, and nothing
is yet.

## 8. Hand over to /implement

Finish with a short summary in the conversation, in the user's language, in
this shape:

- **Goal** — one or two sentences.
- **Decisions** — the settled answers, one line each, with the ADR number where
  one was written.
- **Out of scope** — what was explicitly left out.
- **Test seams** — where the behaviour will be tested, agreed in the rounds.
- **Steps** — the work in the order it should be built.

Then list the files you wrote. Do not start implementing; that is `/implement`,
and the user decides when.
