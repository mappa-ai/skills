# Mappa Conduit SDK Skill

`conduit-sdk` is the canonical agent skill for integrating Mappa Conduit SDKs.

The default path is intentionally simple: upload media, create a report from `media_id`, target the interviewed person with `magic_hint`, and receive the final report by webhook.

Use it when an agent needs to add Conduit to a TypeScript, Python, Go, or Rust codebase, choose the right SDK surface, upload media, target the right speaker, wire the first webhook-first report flow, and stay on documented APIs.

## Name

- Human-facing name: `Mappa Conduit SDK Skill`
- Skill id: `conduit-sdk`
- Recommended slash command: `/conduit-sdk`

Keep the id short and obvious. Top skills win on discoverability, not clever naming.

## What it helps with

- First-time Conduit SDK integrations
- Upload media once and reuse `media_id`
- Select one person from uploaded media with `magic_hint`, for example `the interviewed person`
- Framework-aware webhook handlers for Next.js, FastAPI, `net/http`, and `axum`
- Safe raw-body signature verification
- `reports`-first onboarding with one strong default instead of many branching flows
- Ports and migrations between Conduit SDK languages

## Install With skills.sh

For any agent that supports Agent Skills through `skills.sh`, install the skill from the public skills repo:

```bash
npx skills add https://github.com/mappa-ai/skills --skill conduit-sdk
```

After install, verify it is available in your agent by:

- asking what skills are available
- invoking `/conduit-sdk` directly if your agent exposes slash commands
- giving the agent a first-time Conduit integration prompt and checking that it loads the skill

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

## Included references

- `conduit-sdk/SKILL.md` - routing and integration policy
- `conduit-sdk/references/common.md` - shared onboarding guidance
- `conduit-sdk/references/typescript.md` - TypeScript and Next.js-oriented usage
- `conduit-sdk/references/python.md` - Python and FastAPI-oriented usage
- `conduit-sdk/references/go.md` - Go and `net/http`-oriented usage
- `conduit-sdk/references/rust.md` - Rust and `axum`-oriented usage

## Versioning

The skill version is tracked in `conduit-sdk/SKILL.md` metadata and mirrored from the private source repo.
