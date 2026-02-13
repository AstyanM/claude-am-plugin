# am-plugin

![Claude Code](https://img.shields.io/badge/Claude_Code-Plugin-blueviolet)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Agents](https://img.shields.io/badge/Agents-6-orange)
![Skills](https://img.shields.io/badge/Skills-6-green)
![License](https://img.shields.io/badge/License-MIT-yellow)

A developer toolkit plugin for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — agents, skills, commands, and hooks that turn Claude into a structured engineering assistant.

---

## Motivation

Claude Code is powerful out of the box, but complex engineering workflows benefit from structured guidance.
This plugin provides opinionated agents, reusable skills, and safety hooks so that Claude follows best practices automatically — from architecture design to code review, test generation, and documentation.

---

## Features

| Category | What you get |
|----------|-------------|
| **Architecture & exploration** | Agents that analyze codebases, design feature blueprints, and trace execution flows |
| **Code quality** | Automated code review with confidence scoring, refactoring advisor with smell detection, and performance auditing |
| **Test generation** | Agent that produces idiomatic, runnable test suites matching your project's framework and conventions |
| **API documentation** | Skill that generates OpenAPI 3.1 YAML, Markdown, or Postman collections from actual code |
| **Diagrams** | Skills for DBML database schemas (ERD) and Draw.io diagrams (flowcharts, architecture, UML) |
| **Frontend design** | Skill for distinctive, production-grade UI components that avoid generic AI aesthetics |
| **Documentation** | README generator/editor, CLAUDE.md auditor, and session-based learning capture |
| **Security hook** | Pre-edit hook that detects dangerous patterns (eval, innerHTML, os.system, command injection in GitHub Actions) |
| **Guided workflows** | 7-phase feature development command with discovery, architecture, implementation, and review |

---

## Components

### Agents

| Agent | Description |
|-------|-------------|
| `code-architect` | Designs feature architectures with implementation blueprints |
| `code-explorer` | Traces execution paths and maps architecture layers |
| `code-reviewer` | Reviews code with confidence-based filtering (threshold ≥ 80%) |
| `test-generator` | Generates comprehensive test suites adapted to the project's patterns |
| `refactoring-advisor` | Detects code smells and proposes safe refactoring plans |
| `performance-auditor` | Identifies bottlenecks (N+1, re-renders, memory leaks) with P0–P2 scoring |

### Skills

| Skill | Description |
|-------|-------------|
| `api-docs` | Generates API documentation (OpenAPI, Markdown, Postman) from code |
| `dbml` | Creates database schemas in DBML, visualizable as ERD in VS Code |
| `drawio` | Creates Draw.io diagrams in XML (flowcharts, architecture, UML) |
| `frontend-design` | Builds distinctive, production-grade frontend interfaces |
| `readme-editor` | Generates and edits structured, high-quality README files |
| `claude-md-improver` | Audits and improves CLAUDE.md files with quality scoring |

### Commands

| Command | Description |
|---------|-------------|
| `feature-dev` | Guided feature development in 7 phases (discovery → summary) |
| `readme-generate` | Generates a complete README from the actual codebase |
| `revise-claude-md` | Updates CLAUDE.md with learnings from the current session |

### Hooks

| Hook | Trigger | Description |
|------|---------|-------------|
| `security_reminder_hook` | `PreToolUse` on Edit/Write | Warns about security anti-patterns (eval, innerHTML, command injection, pickle, os.system, etc.) — once per rule per file per session |

---

## Project Structure

```
am-plugin/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── agents/
│   ├── code-architect.md
│   ├── code-explorer.md
│   ├── code-reviewer.md
│   ├── performance-auditor.md
│   ├── refactoring-advisor.md
│   └── test-generator.md
├── commands/
│   ├── feature-dev.md
│   ├── readme-generate.md
│   ├── readme.md
│   └── revise-claude-md.md
├── hooks/
│   ├── hooks.json
│   └── security_reminder_hook.py
├── skills/
│   ├── api-docs/
│   ├── claude-md-improver/
│   ├── dbml/
│   ├── drawio/
│   ├── frontend-design/
│   └── readme-editor/
├── mcp.json                     # Serena MCP server config
├── LICENSE
└── README.md
```

---

## Installation

```bash
claude plugin add /path/to/am-plugin
```

Or reference it in your `.claude/settings.json`:

```json
{
  "plugins": ["/path/to/am-plugin"]
}
```

---

## MCP Server

The plugin includes a [Serena](https://github.com/oraios/serena) MCP server configuration for semantic code analysis (symbol navigation, referencing, renaming). It is configured in `mcp.json` and requires `uvx` to be available.

---

## Author

- **Martin Astyan** — [GitHub](https://github.com/AstyanM)

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
