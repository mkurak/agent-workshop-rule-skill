# Agent Configuration Rules

## Children Pattern (Mandatory)

Every agent is organized in the following structure:

```
~/.claude/agents/{agent-name}/
├── agent.md              ← Identity, area of responsibility, core principles (short, embedded)
└── children/             ← Detailed information, patterns, strategies (each topic in a separate file)
    ├── konu-1.md
    ├── konu-2.md
    └── ...
```

### Rules

1. **agent.md stays short.** Only: identity, area of responsibility (positive list), core principles (unchanging, short bullet points), "read children/" instruction.
2. **Everything detailed goes under children/.** Strategies, patterns, workflows, conventions -- each in a separate .md file.
3. **New topic = new file.** Without touching agent.md, add a .md file under children/. The agent auto-discovers it via the "read all .md files under children/" instruction.
4. **Update = single file.** To update a topic, only the relevant children file is touched.
5. **Monolithic agent files are prohibited.** Piling all information into a single .md is prohibited -- it becomes unmanageable.
6. **This pattern applies to all agents.** API, Socket, Worker, Flutter, React, Mail, Log, Infra -- all follow the same structure.
