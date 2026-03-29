# OpenClaw Skills

A collection of workspace skills for [OpenClaw](https://github.com/openclaw/openclaw), the open-source AI agent framework.

## Available Skills

| Skill | Description |
|---|---|
| [spring-clean](./spring-clean/) | Audit workspace files for token bloat, cross-file conflicts, redundancy, stale content, and misplaced information. Produces a diff-preview for human approval before any writes. |

## What are workspace files?

In agent frameworks like OpenClaw, Claude Code, or Codex, files like `AGENTS.md`, `TOOLS.md`, and `USER.md` give the LLM persistent instructions, preferences, and domain knowledge. They're injected into every API call as part of the system prompt.

The downside: they make every prompt more expensive and eat into your context window. These skills help you manage that tradeoff.

## Installation

**Via ClawHub (recommended):**

```bash
clawhub install al-matty/spring-clean
```

**Manual:**

Copy the skill folder into your OpenClaw workspace:

```bash
cp -r spring-clean ~/.openclaw/workspace/skills/
```

The skill will be available on your next session.

## Context

These skills were born out of running OpenClaw as a daily-driver personal assistant. Blog posts with more background:

- *Coming soon: workspace file management for AI agents*

## License

MIT
