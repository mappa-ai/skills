# refine-linear-issue

## What it is

A workflow skill that turns a **rough problem or feature idea** into a **structured Linear issue** grounded in the repository: title, context, proposed solution, acceptance criteria, and optional technical notes and out-of-scope boundaries. It supports **creating** a new issue from a plain description or **editing** an existing issue when you pass a Linear issue ID or URL.

## When to use it

Use when you want the agent to:

- Draft or sharpen a Linear ticket using real file and pattern references from the codebase
- Update an existing issue’s body with a clearer structure while keeping assignee, status, and labels intact (when using edit mode)

**When not to use it:** When you only need a quick note without codebase grounding, or when Linear tools (`get_issue`, `save_issue`) are not available to the agent.

## What the agent will do

Understand the input (create vs edit), explore the repo for relevant files and patterns, ask 1–2 clarifying questions only if needed, then produce the structured sections above. In **edit mode** it fetches the issue and applies the refinement directly. In **create mode** it shows the draft for approval before creating. Full behavior is in [SKILL.md](SKILL.md).

## Prerequisites

- **Linear integration** exposed to the agent (e.g. MCP or equivalent) implementing issue fetch/update/create as referenced in [SKILL.md](SKILL.md) (`get_issue`, `save_issue`).
- Repository access so the agent can search and cite code accurately.

## How to install

```bash
npx skills add https://github.com/mappa-ai/skills --skill utils-skills/refine-linear-issue
```

## Example prompts

```text
Turn this into a Linear issue: the export button drops rows when the table has more than 10k rows — investigate pagination in the reports module.
```

```text
Refine Linear issue ABC-123 with what we learned: include file paths and acceptance criteria for the webhook retry change.
```

```text
Create a Linear ticket for adding dark mode to the settings page; use our existing theme token pattern from the design system package.
```

## Files in this folder

| File | Purpose |
|------|---------|
| [SKILL.md](SKILL.md) | Agent workflow: explore, draft structure, create vs edit rules |
| [README.md](README.md) | This human-facing overview |

## Maintenance

- Keep tool names in [SKILL.md](SKILL.md) aligned with your Linear MCP (or adapter) if APIs differ.
- Adjust the `description` in [SKILL.md](SKILL.md) frontmatter if you add new triggers (e.g. “Linear task”, project-specific wording).
