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

Look at the project's `.claude/docs/coding-standards/` directory and identify existing app files.

| Scope | File |
|-------|------|
| Common to all applications | `.claude/rules/coding-common.md` |
| A specific application | `.claude/docs/coding-standards/{app}.md` (select from existing files) |

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

### 7. Git Push for Team Scope
For rules written with `--team`:
```bash
cd ~/.claude/repos/agentteamland/{team-name}
git add -A
git commit -m "rule: {kebab-case-rule-id}"
git push
```

---

## Important Rules

1. **Language:** The user may invoke the skill in any language; the skill always writes the rule in English.
2. **Ask if information is missing.** Never fill in gaps on your own.
3. **Do not create duplication.** Read existing rules first.
4. **Validate file paths.** If you determine the scope incorrectly, the rule goes to the wrong file.
5. **No format deviations.** All required fields must be filled: Rule, Why, Apply when, Examples.
6. **Automatic git push for team scope.** Commit + push after the rule is written.
