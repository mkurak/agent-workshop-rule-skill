---
name: rule
description: "Add a new coding/architecture rule. The user describes it in natural language (any language), the skill writes it in English structured format to the correct file. 3 scopes: project (default), --global, --team."
argument-hint: "[--global|--team] <rule in natural language, any language>"
---

# /rule Skill

Analyzes a rule expressed in natural language (any language) and writes it to the correct file in English structured format.

---

## Three Scopes

| Flag | Target | When |
|------|--------|------|
| *(none)* | Project `.claude/` directory | Rules specific to this project (default) |
| `--global` | `~/.claude/rules/` | Personal rules that apply to every project |
| `--team` | `~/.claude/repos/agentteamland/{team}/` files | Agent or rule files in the team repo |

**`--team` active team detection:**
- The `readlink` result of symlinks under `~/.claude/agents/` is used to extract `~/.claude/repos/agentteamland/{team-name}/`
- If there is only one team, it is used automatically
- If there are multiple teams, the user is asked via AskUserQuestion

---

## Flow

### 1. Analyze the Rule
Extract the following from the user's natural-language expression:
- **Topic:** What type of rule? (coding, architecture, naming, error handling, etc.)
- **Scope:** Which application(s) does it affect?
- **Motivation:** Why this rule? (If not explicitly stated, derive a reasonable Why -- if unsure, ask.)

### 2. Determine the Target File

**Project scope (default):**

Look at the project's `.atl/docs/coding-standards/` directory and identify existing app files.

| Scope | File |
|-------|------|
| Common to all applications | `.claude/rules/coding-common.md` |
| A specific application | `.atl/docs/coding-standards/{app}.md` (select from existing files) |

**Global scope (`--global`):**

| Scope | File |
|-------|------|
| General rule | `~/.claude/rules/{topic}.md` (append to existing file, or create if none exists) |

**Team scope (`--team`):**

The target is determined based on what the rule pertains to:

| Related area | File |
|-------------|------|
| An agent's knowledge base | `~/.claude/repos/agentteamland/{team}/agents/{agent}.md` |
| Team-wide rule | `~/.claude/repos/agentteamland/{team}/rules/{topic}.md` |

If it applies to more than one but not all, ask the user.

### 3. Check Existing Rules
**Always read** the target file. Three situations are possible:
- **Entirely new rule:** Add as a new section
- **Extending/updating an existing rule:** Update in-place, do not create duplication
- **Conflict:** Two rules contradict each other -- ask the user, do not assume

### 4. Write in Structured Format
In English, **detailed and clear** -- not brief. An incomplete rule is more dangerous than no rule at all.

```markdown
### {kebab-case-rule-id}
**Rule:** {Clear statement of the rule in a single sentence}

**Why:** {Motivation. What problem does it prevent? What principle does it support?
Include lessons from past mistakes if applicable. This field must not be left empty or vague.}

**Apply when:** {Under what circumstances does it apply -- file paths, code patterns,
what types of changes? Be specific.}

**Don't apply when:** {(Optional) Explicitly state exceptions if any.}

**Examples:**
- ✅ Correct: {code example or concrete scenario}
- ❌ Wrong: {code example or concrete scenario}

**Related:** {(Optional) Related rule IDs}
```

### 5. Writing Rules (CRITICAL)
- **Never assume.** If information is missing, ask.
- **Do not keep it short -- explain.** Skipped detail = unenforced rule.
- **Capture edge cases.** Add `Don't apply when`.
- **Provide examples.** Both ✅ and ❌.
- **Assign a unique ID.** Read the file first to avoid conflicts.

### 6. Write and Verify
- Update the target file with Edit.
- Give the user a brief summary: which file and which ID it was written to.

### 7. Persisting Team-Scope Rules
For rules written with `--team`, the rule file lives under the team's local clone. Every public `agentteamland/{team}` repo is branch-protected, so direct push to `origin/main` is refused. Open a PR instead:

```bash
cd ~/.claude/repos/agentteamland/{team-name}
git checkout -b rule/{kebab-case-rule-id}
git add rules/{file}.md team.json
git commit -m "rule: {kebab-case-rule-id}"
git push -u origin rule/{kebab-case-rule-id}
gh pr create --fill
```

The `/create-pr` skill in `core` automates this if installed.

---

## Important Rules

1. **Language:** The user may invoke the skill in any language; the skill always writes the rule in English.
2. **Ask if information is missing.** Never fill in gaps on your own.
3. **Do not create duplication.** Read existing rules first.
4. **Validate file paths.** If you determine the scope incorrectly, the rule goes to the wrong file.
5. **No format deviations.** All required fields must be filled: Rule, Why, Apply when, Examples.
6. **Team-scope rules ship via PR, not direct push.** Every public `agentteamland/{team}` repo is branch-protected; the skill writes the rule file to the team's local clone and instructs the user to open a PR (or use `/create-pr`).

## Accumulated Learnings

(Auto-rebuilt by /save-learnings from learnings/*.md frontmatter. Do not edit by hand. Currently empty — populates as the skill is used and edge-case learnings accumulate.)
