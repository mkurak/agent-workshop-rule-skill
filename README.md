# 📏 Rule Skill

Claude Code skills for managing coding rules and standards. Write rules in natural language, get structured, enforceable rule definitions. Includes a guided wizard for complex rules.

## Installation

```bash
atl install rule
```

> Requires the [`atl` CLI](https://github.com/agentteamland/cli) to be installed first.

## Usage

### Quick Rule Addition
```bash
# Describe a rule in natural language (any language — output is always English)
/rule API controllers should not write try-catch; global handler takes over
```

The skill analyzes your input, determines the correct file, checks for duplicates, and writes a structured rule in English.

### Guided Wizard (for complex rules)
```bash
# Multi-turn Q&A to refine a rule before writing
/rule-wizard Logging usage in controllers
```

The wizard asks multiple-choice questions to clarify scope, motivation, examples, and edge cases before generating the final rule.

### Three scopes — `--global` and `--team` extend beyond the current project

Both `/rule` and `/rule-wizard` accept an optional scope flag:

| Flag | Target | When |
|---|---|---|
| *(none)* | The current project's `.claude/` files | Project-specific rules (default) |
| `--global` | `~/.claude/rules/{topic}.md` | Personal rules that apply to every project |
| `--team` | `~/.claude/repos/agentteamland/{team}/` agent or rule files | Rules contributed back to a team repo (ships via PR per the team-repo-maintenance discipline) |

```bash
# Personal global rule that applies to every project
/rule --global Always run prettier on .ts and .tsx files before committing

# Team-scope rule (asked which agent's knowledge base to add to, or as a team-wide rule)
/rule-wizard --team Worker should never connect to DB directly
```

For `--team`, active team is auto-detected from installed `.claude/agents/` symlinks. Single team → used automatically; multiple teams → asked via `AskUserQuestion`. Team-scope rules are written to the team's local clone and require a PR to land — the skill instructs you on the PR command (or use [`/create-pr`](https://github.com/agentteamland/core/blob/main/skills/create-pr/skill.md) for the automated path).

## What's Included

| Type | File | Purpose |
|------|------|---------|
| Skill | `skills/rule/skill.md` | The `/rule` command — direct rule writing |
| Skill | `skills/rule-wizard/skill.md` | The `/rule-wizard` command — guided Q&A |

> Note: `agent-structure.md` rule (cross-team agent + skill structure) ships in [`agentteamland/core`](https://github.com/agentteamland/core), not here.

## Rule Format

Every rule follows a structured template:

```markdown
### kebab-case-rule-id
**Rule:** Single sentence statement.

**Why:** Motivation — what problem does this prevent?

**Apply when:** Specific conditions, file paths, code patterns.

**Don't apply when:** (Optional) Exceptions.

**Examples:**
- ✅ Correct: code example
- ❌ Wrong: code example

**Related:** (Optional) Related rule IDs
```

## How Rules Are Organized

Per-scope file resolution. The `/rule` skill automatically determines which file to write to.

### Project scope (default — no flag)

Two locations inside the current project:

| Sub-scope | Location | When Loaded |
|-------|----------|-------------|
| Cross-cutting | `.claude/rules/coding-common.md` | Always (every conversation) |
| App-specific | `.claude/docs/coding-standards/{app}.md` | Auto-injected when editing files in that app directory |

### Global scope (`--global`)

| Sub-scope | Location | When Loaded |
|---|---|---|
| Personal cross-project rule | `~/.claude/rules/{topic}.md` | Always (every conversation across every project) |

### Team scope (`--team`)

| Sub-scope | Location | When Loaded |
|---|---|---|
| Agent-specific knowledge | `~/.claude/repos/agentteamland/{team}/agents/{agent}.md` | When that agent is invoked in any project that has the team installed |
| Team-wide rule | `~/.claude/repos/agentteamland/{team}/rules/{topic}.md` | Always for any project with the team installed |

Team-scope writes go to the team's local clone first; landing them upstream requires a PR (see the [team-repo-maintenance rule](https://github.com/agentteamland/core/blob/main/rules/team-repo-maintenance.md)).

## Key Features

- **Natural language input** — describe rules in any language, skill writes in English
- **Duplicate detection** — reads existing rules before writing
- **Conflict detection** — warns if new rule contradicts an existing one
- **Dynamic scope** — scans `coding-standards/` directory to discover available apps
- **Multi-rule detection** — wizard identifies when input contains multiple rules

## License

MIT
