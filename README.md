# Mindflow skills

Two agent skills for Mindflow users.

**`mindflow-feasibility`** interviews someone about an automation use case and tells them
whether Mindflow can do it: a feasibility score based only on API access and
connectivity, the build effort (kept separate), a flow sketch, open points, and an offer
to email the report.

**`mindflow-assistant`** helps people use the Mindflow Studio: it explains
errors and run logs, walks through the canvas, triggers, credentials, environments, versions
and fx functions, and gives ready-made flow patterns (approvals, polling, pagination,
fallbacks, parallel processing, business hours, private endpoints).

Both work with any agent that reads `SKILL.md` skills, such as Claude and Codex.

## Contents

```
mindflow-feasibility/
  SKILL.md          interview flow, scoring rules, safety
  references/       scoring, capabilities, interview guide, report and email
mindflow-assistant/
  SKILL.md          role, safety rules, where to look
  references/       troubleshooting, steps, fx functions, triggers, credentials,
                    environments and versions, runs and logs, AI agents, patterns,
                    missing/null/large data, private endpoints, driving the Studio
```

## Install

- **Claude Code**: copy a skill folder into `~/.claude/skills/` (or a project's
  `.claude/skills/`).
- **Claude apps**: zip a skill folder and upload it as a skill.
- **Codex**: copy a skill folder into your Codex skills folder (for example
  `~/.codex/skills/`).

## Notes

The Studio changes with releases: when a label differs from the skill, trust the screen.
