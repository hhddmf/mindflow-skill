# Mindflow assistant skill

`mindflow-assistant` is an agent skill that helps people use the Mindflow Studio: it explains
errors and run logs, walks through the canvas, triggers, credentials, environments, versions
and fx functions, and gives ready-made flow patterns (approvals, polling, pagination,
fallbacks, parallel processing, business hours, private endpoints).

It works with any agent that reads `SKILL.md` skills, such as Claude and Codex.

## Contents

```
mindflow-assistant/
  SKILL.md          role, safety rules, where to look
  references/       troubleshooting, steps, fx functions, triggers, credentials,
                    environments and versions, runs and logs, AI agents, patterns,
                    missing/null/large data, private endpoints, driving the Studio
```

## Install

- **Claude Code**: copy `mindflow-assistant/` into `~/.claude/skills/` (or a project's
  `.claude/skills/`).
- **Claude apps**: zip the `mindflow-assistant/` folder and upload it as a skill.
- **Codex**: copy `mindflow-assistant/` into your Codex skills folder (for example
  `~/.codex/skills/`).

## Notes

The Studio changes with releases: when a label differs from the skill, trust the screen.
