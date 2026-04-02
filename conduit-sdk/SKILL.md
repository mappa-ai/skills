---
name: conduit-sdk
description: Use this skill when integrating Mappa Conduit SDKs from TypeScript, Python, Go, or Rust. It helps choose the right SDK surface, upload media, target the right speaker, build the first webhook-first report flow, and use matching or primitives only when the task explicitly needs them.
metadata:
  author: mappa-ai
  version: "0.2.2"
---

## Role

- Treat this skill as the canonical public Conduit integration guide for agents.
- Optimize for first successful integration, not exhaustive SDK tours.
- Use only the documented surfaces in the bundled references. If a method, type, or request shape is not shown there, say so and do not invent it.

## How to load references

1. Read `references/common.md` first.
2. Detect the target language before giving code:
    - Explicit user language or framework wins.
    - Otherwise inspect repo manifests near the task.
3. Read only one of `references/typescript.md`, `references/python.md`, `references/go.md`, or `references/rust.md`.
4. Read multiple language references only for ports, comparisons, or polyglot repos.
5. If the repo still points to multiple equally plausible SDK languages after inspection, ask one targeted language question.

## Framework and workflow defaults

- If the repo or request implies a framework, answer in that framework: Next.js for TypeScript, FastAPI for Python, `net/http` for Go, `axum` for Rust.
- Otherwise stay framework-agnostic and use language-native SDK examples.
- Default to `reports` unless the user explicitly asks for target-versus-group comparison.
- First answers should show one obvious path: upload media, create a report from `media_id`, target the speaker with `magic_hint`, add the webhook handler, verify the signature on the raw body, dedupe on `event.id`, fetch the completed report.

- Prefer webhooks as the default completion path in production. Conduit jobs are asynchronous and often take around 150 seconds.
- Use receipt handles like `wait()` or `stream()` only for local scripts, operator tooling, or explicit synchronous control.
- Reach for `primitives.media`, `primitives.jobs`, and `primitives.entities` only when the user needs upload reuse, job inspection or cancellation, or stable entity management.
- When the user needs to pick one person from uploaded media, prefer `magic_hint` first with a concrete hint like `the interviewed person`.

## Conduit-specific guardrails

- Verify webhook signatures on the exact raw request body before parsing the event.
- Deduplicate webhook deliveries on `event.id` before doing side effects.
- Keep onboarding examples on the stable surface unless the user explicitly asks for lower-level control.
- Preserve idempotency keys when the caller already has a stable external job identifier.
- Match package names, imports, and method names exactly from the chosen language reference.
- If the user asks for browser TypeScript, call out the `dangerouslyAllowBrowser` requirement.

## First-time integration checklist

1. Initialize the client with the correct package import and API key shape.
2. Upload the recording and reuse the returned `media_id`.
3. Create a `reports` job with `target = magic_hint("the interviewed person")`.
4. Add the webhook route in the detected framework.
5. Verify the signature on the exact raw request body before parsing.
6. Deduplicate on `event.id`.
7. Handle `report.completed` and fetch the final report.
8. Mention `wait()` or `stream()` only as local tooling helpers when relevant.

## When to branch out

- Use `matching` only for explicit target-versus-group comparisons in a named context.
- Use `primitives.media` when the recording should be uploaded once and reused by `media_id`.
- Use `primitives.jobs` for job inspection or cancellation.
- Use `primitives.entities` for stable entity management or backoffice flows.
- Use direct `entity_id` targeting only when the caller already knows the stable speaker identity.

## Output expectations

- Keep first examples short: upload media, create a report from `media_id`, target with `magic_hint`, verify a webhook, dedupe, fetch the final report.
- Use the user language and framework idioms from the chosen reference instead of translating through TypeScript first.
- If the user is migrating between SDKs, map equivalent surfaces directly instead of re-explaining Conduit from scratch.
- Do not lead with `matching`, primitives, polling, or architecture talk when the user just needs the first integration working.
- If you are tuning this skill itself, use `assets/trigger-queries.json` as the trigger eval fixture set.
