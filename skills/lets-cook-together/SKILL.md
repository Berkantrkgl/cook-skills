---
name: lets-cook-together
description: A relentless design interview that sharpens a plan until we share one understanding of it — round by round, a decision tree, the project's language made exact on the way — and ends with a summary in the conversation that /implement builds from. Writes no files; recording is /update-project-docs's job, after the work exists.
disable-model-invocation: true
---

# Let's cook together

Interview the user about a plan, a design or an idea until the two of you reach
a **shared understanding** of it — every detail settled, no open point left.
Then hand it over to be built, as a summary in the conversation.

**Language.** Talk in whatever language the user is writing in, and keep to it
for the whole session.

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

## 5. Nothing is written — not during the session, not after it

This skill writes **no file**: no ADR, no glossary entry, no `state.md` row,
no code. It ends in the conversation. Recording a decision before anything is
built means recording an intention, and an intention changes the moment the
user sees the result — a record written now has to be superseded before it was
ever true. The docs are written by `/update-project-docs`, from what was
actually built. Keep the settled decisions and resolved terms in the
conversation, where `/implement` reads them.

## 6. Ending

**The normal end:** the frontier is empty — every branch visited, nothing
silently assumed. Say so, and ask the user to confirm that you have reached a
shared understanding.

**The early end:** the user says to stop ("enough", "go ahead", "uygula" or the
like). Stop asking immediately. Show, in one list, the decisions still open and
what you are assuming for each, and ask **one** confirmation: *these are my
assumptions — agreed?* Do not reopen the rounds unless the user rejects one.

## 7. The summary

After the confirmation, finish with the summary `/implement` builds from, in
the user's language, in this shape:

- **Goal** — one or two sentences.
- **Decisions** — the settled answers, one line each. Specific enough to build
  from without asking again: names, routes, labels, rules, edge cases.
- **Terms** — the project words this session made exact, one line each, if any.
- **Out of scope** — what was explicitly left out.
- **Steps** — the work in the order it should be built.

The summary is the hand-over and the only record. Do not start implementing;
that is `/implement`, and the user decides when.
