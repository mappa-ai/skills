# Conduit SDK common reference

## Purpose

Use this file as the canonical public onboarding router for agents. The job is to get the user's first production-safe Conduit integration working with the fewest moving parts.

## First-time integration default

- Default to `reports` for the first integration.
- Default to upload once, reuse `media_id`, then create the report.
- Default to `magic_hint` when the user needs one person selected from uploaded media.
- Prefer a webhook-first flow in production.
- Treat `report.completed` as the happy-path event unless the user explicitly needs `matching`.
- Keep the first answer focused on one media upload, one report create call, and one webhook handler.
- Mention `wait()` or `stream()` only as local script or operator tooling helpers.

## Mental model

Conduit has two primary stable workflows:

- `reports` analyzes one selected speaker from a recording.
- `matching` compares one target against a group inside a named interpretation context.

Create calls are receipt-first and asynchronous. They resolve source handling, upload when needed, accept the job, and return a receipt. They do not wait for the final report or matching result.

Canonical naming is semantic, not language-specific. Keep `receipt`, `report.completed`, `matching.completed`, `general_report`, `sales_playbook`, and `behavioral_compatibility` intact across languages. Casing may change to fit the language, but business names do not.

## Workflow chooser

- Use `reports.create(...)` for one-speaker analysis and for the default first integration.
- Use `matching.create(...)` when the user wants a target-versus-group comparison.
- Treat `behavioral_compatibility` as the current stable matching context unless the product spec changes.
- Use `primitives.media.upload(...)` when the caller wants to upload once and reuse a `media_id` later.
- Use `primitives.jobs.get(...)` or cancel helpers only when the caller needs job inspection or cancellation.
- Use `primitives.entities.*` only for stable entity management and backoffice flows.

## Language and framework selection

- Explicit user language or framework wins.
- Otherwise inspect the repo and follow the obvious stack.
- If the repo implies a framework, use that framework's request handler and raw-body access pattern.
- If language is still ambiguous after inspection, ask one targeted language question.

## Default production path

- Prefer webhooks as the completion path.
- Verify the webhook signature on the raw body before parsing the event.
- Deduplicate deliveries on `event.id` because delivery is at least once.
- Fetch the final report or matching object after receiving `*.completed`.

## First webhook order

1. Read the exact raw request body.
2. Verify the signature on that raw body.
3. Parse the event.
4. Return early for duplicate `event.id`.
5. Handle `report.completed` for the first integration.
6. Fetch the final report with `reports.get(...)`.
7. Acknowledge success.

## Local and operator paths

- `handle.wait(...)` is a convenience for scripts, local development, and operator tooling.
- `handle.stream(...)` is for callers that want live job stage updates.
- Do not present wait or stream as the default production integration unless the user explicitly wants polling.

## Stable surface bias

Default to the stable public surface:

- reports create/get
- webhooks verify + parse
- matching create/get
- primitives media/entities/jobs only when needed

Avoid inventing hidden helper layers when a direct SDK call is enough.
If the selected reference does not show a method or shape, say so and keep the answer on documented surfaces.

## Shared source guidance

Across SDKs, the common source shapes are:

- existing uploaded media by `media_id`
- in-memory bytes or file content
- remote `url`
- filesystem `path` when the runtime supports it

Source validation is a one-of contract. Do not mix variants in one request.

Default recommendation:

- If the user is doing a first integration, show `primitives.media.upload(...)` first.
- Reuse the returned `media_id` in `reports.create(...)`.
- Keep `source.url` and `source.path` documented as alternate inputs, not the main onboarding path.

Important behavior:

- `source.url` means the SDK host runtime fetches the remote media and then uploads it to Conduit.
- `source.path` resolves relative to the current working directory in filesystem-capable runtimes.
- Upload-capable sources may materialize payloads in memory before submission.
- Receipt timing is `resolve source -> validate size/runtime -> upload -> create job -> return receipt`.

## Shared target guidance

Across SDKs, the common report target selectors are:

- `magic_hint` for the common case: pick one described person from uploaded media
- `dominant` for best-effort default speaker selection
- `entity_id` when the caller already knows the stable speaker identity
- `timerange` only when the user has explicit time bounds

Default recommendation:

- If the user describes a role or person in uploaded media, prefer `magic_hint` first.
- Use a concrete hint in the example, such as `the interviewed person`.
- Keep `entity_id` as the follow-up path once the caller already has a stable entity.

## Common implementation defaults

- Keep examples webhook-first.
- Keep first examples short and production-safe.
- Keep the canonical first example on `upload -> media_id -> magic_hint -> webhook`.
- Mention idempotency keys when the caller has an external job identifier.
- Show `event.id` dedupe in webhook examples.
- Keep advanced primitives out of onboarding examples.
- Use stable method names from the language-specific reference exactly.
