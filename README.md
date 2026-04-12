# 📏 Rule Skill

Claude Code skills for managing coding rules and standards. Write rules in natural language, get structured, enforceable rule definitions. Includes a guided wizard for complex rules.

## Installation

```bash
/team install https://github.com/mkurak/agent-workshop-rule-skill.git
```

> Requires [Agent Team Manager](https://github.com/mkurak/agent-workshop-agent-team-manager-skill) to be installed first.

## Usage

### Quick Rule Addition
```bash
# Describe a rule in natural language (Turkish or English)
/rule API endpoint'lerinde try-catch yazılmasın, global handler üstlensin
```

The skill analyzes your input, determines the correct file, checks for duplicates, and writes a structured rule in English.

### Guided Wizard (for complex rules)
```bash
# Multi-turn Q&A to refine a rule before writing
/rule-wizard Controller'larda logging kullanımı
```

The wizard asks multiple-choice questions to clarify scope, motivation, examples, and edge cases before generating the final rule.

## What's Included

| Type | File | Purpose |
|------|------|---------|
| Skill | `skills/rule/skill.md` | The `/rule` command — direct rule writing |
| Skill | `skills/rule-wizard/skill.md` | The `/rule-wizard` command — guided Q&A |
| Rule | `rules/agent-structure.md` | Agent organization standard (children pattern) |

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

Rules live in two locations per project:

| Scope | Location | When Loaded |
|-------|----------|-------------|
| Cross-cutting | `.claude/rules/coding-common.md` | Always (every conversation) |
| App-specific | `.claude/docs/coding-standards/{app}.md` | Auto-injected when editing files in that app directory |

The `/rule` skill automatically determines which file to write to based on the rule's scope.

## Key Features

- **Natural language input** — describe rules in Turkish, skill writes in English
- **Duplicate detection** — reads existing rules before writing
- **Conflict detection** — warns if new rule contradicts an existing one
- **Dynamic scope** — scans `coding-standards/` directory to discover available apps
- **Multi-rule detection** — wizard identifies when input contains multiple rules

## License

MIT
