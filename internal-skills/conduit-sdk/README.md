# Mappa Conduit SDK Skill

## What it is

The canonical agent skill for integrating **Mappa Conduit SDKs** from **TypeScript, Python, Go, or Rust**. It optimizes for a first successful integration: upload media, create a report from `media_id`, target the right speaker with `magic_hint`, complete the flow via **webhooks** with raw-body signature verification, and stay on **documented APIs only** (see bundled references).

## When to use it

Use when an agent (or you driving an agent) needs to:

- Add Conduit to an app in one of the supported languages
- Choose the right SDK surface and framework defaults (e.g. Next.js, FastAPI, `net/http`, `axum`)
- Wire webhook-first completion, dedupe on `event.id`, and fetch the final report

**When not to use it:** When the task is unrelated to Conduit, or when you need behavior not covered in the reference docs—in that case the skill instructs the agent to say so instead of inventing APIs.

## What the agent will do

Load `references/common.md`, then **one** language reference unless porting or comparing. Default to the `reports` flow and webhook completion; mention synchronous helpers only for scripts or explicit requests. Full policy, checklists, and guardrails are in [SKILL.md](SKILL.md).

## Prerequisites

- Mappa Conduit API credentials and SDK packages as appropriate for the target language
- A reachable webhook endpoint for production-style flows
- Familiarity with your framework’s raw-body handling for signature verification

## How to install

For agents that support Agent Skills via `skills.sh`, install from this repository using the **nested skill path**:

```bash
npx skills add https://github.com/mappa-ai/skills --skill internal-skills/conduit-sdk
```

If your tooling only supports a flat skill directory layout, install from a revision or layout that exposes `conduit-sdk` at the root, or vendor this folder into your project’s skills directory.

After install:

- Ask the agent what skills are available, or invoke `/conduit-sdk` if your agent exposes slash commands
- Run a small “first integration” prompt and confirm it loads language-specific references

## Example prompts

```text
Add Mappa Conduit to my FastAPI app. Upload the interview recording, target the interviewed person, and show the verified webhook flow.
```

```text
Add Mappa Conduit to my Next.js app. Upload the interview recording, target the interviewed person, and fetch the final report from the webhook flow.
```

```text
Port this Conduit TypeScript integration to Rust axum and keep the webhook-first flow.
```

```text
I already upload recordings separately. Show me how to create a Conduit report from media_id in Python and target the interviewed person.
```

```text
Compare one candidate against a hiring panel in Conduit using Go and keep the stable matching context.
```

## Files in this folder

| Path | Purpose |
|------|---------|
| [SKILL.md](SKILL.md) | Routing, defaults, checklists, trigger eval note |
| [references/common.md](references/common.md) | Shared onboarding |
| [references/typescript.md](references/typescript.md) | TypeScript / Next.js |
| [references/python.md](references/python.md) | Python / FastAPI |
| [references/go.md](references/go.md) | Go / `net/http` |
| [references/rust.md](references/rust.md) | Rust / `axum` |
| [assets/trigger-queries.json](assets/trigger-queries.json) | Trigger evaluation fixtures |
| [README.md](README.md) | This human-facing overview |

## Versioning

The skill version is recorded in [SKILL.md](SKILL.md) frontmatter under `metadata.version` and may be mirrored from a private source repository.

## Name and id

- Human-facing name: **Mappa Conduit SDK Skill**
- Skill id: `conduit-sdk`
- Suggested slash command: `/conduit-sdk`
