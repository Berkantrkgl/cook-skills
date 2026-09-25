# cook-skills

Four skills for building software with an AI coding agent. They work with
Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot and any other agent that
reads `SKILL.md` files.

| skill | what it does | invoked |
|---|---|---|
| `lets-cook-together` | Interviews you about a design, a few short questions per round, until you both understand it the same way. It sharpens the project's vocabulary along the way. When you confirm, it records the decisions (ADRs, glossary, `state.md`) and writes a summary `implement` can build from. | by you |
| `implement` | Builds the agreed work test-first at agreed seams. It reviews the result on two separate axes (the project's rules, and what was agreed), updates the docs, and proposes a commit. It never commits without your approval. | by you |
| `init-project-docs` | Sets up the documentation layout below, or migrates an existing pile of docs into it. | by you or the agent |
| `update-project-docs` | Runs after a piece of work ships and at the end of each session. It decides whether the work earns a record, writes or supersedes ADRs, and cleans up `state.md`. | by you or the agent |

## The flow

```
lets-cook-together  →  implement  →  update-project-docs
   (decide)             (build)       (runs inside implement, and at session end)
```

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

With the [skills CLI](https://github.com/vercel-labs/skills), for every agent
on the machine:

```bash
npx skills add Berkantrkgl/cook-skills -g
```

Drop `-g` to install into the current project only. Or copy
`skills/<name>/` into your agent's skills directory by hand, for example
`~/.claude/skills/` for Claude Code.

## Credits

The interview method in `lets-cook-together`, and the test discipline and
two-axis review in `implement`, are adapted from Matt Pocock's
[skills](https://github.com/mattpocock/skills) (MIT).

## License

MIT
