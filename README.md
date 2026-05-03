# 📏 Rule + Rule-Wizard Skills

> Write coding/architecture rules in natural language, get structured English-format rules in the right file. The wizard handles ambiguous rules through option-based Q&A rounds.

`/rule` analyzes a natural-language rule statement (any input language) and writes it to the correct file in English structured format. `/rule-wizard` walks through option-based Q&A rounds before invoking `/rule` — for the cases where you know the area but not the exact wording, the exception, or whether you actually have one rule or two.

Three scopes: project (default), `--global` (personal cross-project rules under `~/.claude/rules/`), `--team` (rules contributed back to a team repo via PR).

## 📚 Documentation

Full docs live at **[agentteamland.github.io/docs](https://agentteamland.github.io/docs/)**.

Most relevant sections:

- [`/rule` skill page](https://agentteamland.github.io/docs/skills/rule) — three scopes, target-file matrix, structured-format spec
- [`/rule-wizard` skill page](https://agentteamland.github.io/docs/skills/rule-wizard) — 4-phase Q&A flow + dynamic multiple-rule detection
- [Concepts: Rule](https://agentteamland.github.io/docs/guide/concepts#rule) — what rules are and how they're loaded
- [Install via `atl`](https://agentteamland.github.io/docs/cli/install) — `atl install rule` (the legacy `/team install` was retired in `team-manager@2.0.0`)

## License

MIT.
