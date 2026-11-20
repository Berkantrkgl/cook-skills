# cook-skills

Five skills for building software with an AI coding agent. They work with
Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot and any other agent that
reads `SKILL.md` files.

| skill | what it does | invoked |
|---|---|---|
| `lets-cook-together` | Interviews you about a design, a few short questions per round, until every detail is settled and you both understand it the same way. It sharpens the project's vocabulary along the way. Writes no files: it ends with a summary in the conversation that `implement` builds from. | by you |
| `implement` | Builds that summary as agreed, without reopening it. Writes no new tests, keeps the existing ones green, fixes what breaks, and ends with one general review of the change. No docs, no commit. | by you |
| `test-the-meal` | Writes real tests for what just changed: two sub-agents in parallel, one testing the agreed behaviour at its edges and one trying to break it, never the happy path alone. Each test is proven able to fail; then the full suite runs once. `/test-the-meal trim <path>` slims an overgrown test file, only when you ask. | by you |
| `init-project-docs` | Sets up the documentation layout below, or migrates an existing pile of docs into it. | by you or the agent |
| `update-project-docs` | The last step. Decides whether the work earns a record at all, puts each rule where it belongs (an ADR only rarely, at most one per piece of work), cleans up `state.md`, and proposes one commit for the code, the tests and the docs. | by you |

## The flow

```
lets-cook-together  →  implement  →  test-the-meal  →  update-project-docs
   (decide)             (build)        (test)           (record + commit)
```

Each skill does one job and runs only when you call it; none of them starts
another. You look at the work between `implement` and `test-the-meal`, so the
tests and the docs are written for what you kept, not for a first draft.

## The documentation layout

```
AGENTS.md                  # short: commands, conventions, traps, routing table
.claude/CLAUDE.md          # `@../AGENTS.md`, only if Claude Code is used
.agents/project_docs/
├── architecture.md        # how the system works and what must not break
├── state.md               # what is decided but not built
├── glossary.md            # project-specific terms, when there are any
└── adr/                   # decisions; superseded, never edited or deleted
```

`project_docs/` stays outside version control by default, and there is no
archive file. Text that stops being true is deleted. The reason behind each
choice survives in its ADR.

## Install

With the [skills CLI](https://github.com/vercel-labs/skills). Run it from the
root of the project you are working in:

```bash
npx skills add Berkantrkgl/cook-skills
```

This installs the skills into **that project only**. The CLI asks which agents
to install for.

To make them available in every project on your machine instead:

```bash
npx skills add Berkantrkgl/cook-skills -g
```

To install without the CLI, copy `skills/<name>/` into your agent's skills
directory, for example `.claude/skills/` in a project for Claude Code.

## Credits

The interview method in `lets-cook-together`, and the test discipline in
`test-the-meal`, are adapted from Matt Pocock's
[skills](https://github.com/mattpocock/skills) (MIT).

## License

MIT
