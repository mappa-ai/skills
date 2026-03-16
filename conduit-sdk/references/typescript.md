# TypeScript reference

## Package identity

- Install: `npm install @mappa-ai/conduit`
- Import: `import { Conduit } from "@mappa-ai/conduit"`

## First report

```ts
import { Conduit } from "@mappa-ai/conduit"

const conduit = new Conduit({ apiKey: process.env.CONDUIT_API_KEY! })

const media = await conduit.primitives.media.upload({
  path: "recordings/demo-call.wav",
})

const receipt = await conduit.reports.create({
  output: { template: "general_report" },
  source: { mediaId: media.mediaId },
  target: { strategy: "magic_hint", hint: "the interviewed person" },
  webhook: { url: "https://your-app.com/api/webhooks/conduit" },
})

console.info(media.mediaId, receipt.jobId, receipt.status)
```

## First webhook

```ts
import { Conduit } from "@mappa-ai/conduit"

const conduit = new Conduit({ apiKey: process.env.CONDUIT_API_KEY! })

export async function POST(req: Request): Promise<Response> {
  const payload = await req.text()

  await conduit.webhooks.verifySignature({
    payload,
    headers: Object.fromEntries(req.headers),
    secret: process.env.CONDUIT_WEBHOOK_SECRET!,
  })

  const event = conduit.webhooks.parseEvent(payload)

  if (await hasSeenEvent(event.id)) {
    return new Response("ok", { status: 200 })
  }

  if (event.type === "report.completed") {
    const report = await conduit.reports.get(event.data.reportId)
    await markSeenEvent(event.id)
    await persistReport(report)
  }

  return new Response("ok", { status: 200 })
}
```

## Framework fit

- If the repo uses Next.js App Router, prefer `export async function POST(req: Request)`.
- If the repo uses Express, Fastify, or Hono, preserve the exact raw request body before JSON parsing.
- If the repo does not imply a framework, keep examples on standard `Request`/`Response` shapes.

## Stable surface

- `conduit.reports.create(...)`
- `conduit.reports.get(...)`
- `conduit.matching.create(...)`
- `conduit.matching.get(...)`
- `conduit.webhooks.verifySignature(...)`
- `conduit.webhooks.parseEvent(...)`
- `conduit.primitives.entities.*`
- `conduit.primitives.media.*`
- `conduit.primitives.jobs.*`

## Webhooks

Use this order:

1. `const payload = await req.text()`
2. `await conduit.webhooks.verifySignature({ payload, headers, secret })`
3. `const event = conduit.webhooks.parseEvent(payload)`
4. Deduplicate on `event.id`
5. Handle `report.completed` first
6. Fetch the final resource with `reports.get(...)` or `matching.get(...)`

`verifySignature(...)` expects the raw request body and a plain object for headers.

## Source shapes

Use upload plus `mediaId` as the default onboarding path. These are the other canonical source forms:

- `{ mediaId: string }`
- `{ file: Blob | ArrayBuffer | Uint8Array | ReadableStream<Uint8Array>; label?: string }`
- `{ url: string; label?: string }`
- `{ path: string; label?: string }`

## Target selection

Use `magic_hint` first when the user needs one person selected from uploaded media.

```ts
{ strategy: "magic_hint", hint: "the interviewed person" }
```

Other stable selectors:

- `{ strategy: "dominant" }`
- `{ strategy: "entity_id", entityId: "ent_123" }`
- `{ strategy: "timerange", timeRange: { startSeconds: 15, endSeconds: 45 } }`

## Runtime notes

- Node and Bun support `source.file`, `source.url`, `source.path`, `wait()`, `stream()`, and webhook verification.
- Deno and edge runtimes do not support `source.path`.
- Browser use is blocked by default unless `dangerouslyAllowBrowser: true` is enabled.
- In browser mode, call out that secret API keys should not be exposed unless the user has an explicit trusted browser flow.

## Local wait or stream

Use local helpers only for scripts, development, or operator tooling.

```ts
const report = await receipt.handle?.wait({ timeoutMs: 5 * 60_000 })
```

## Errors

Prefer the SDK helpers for typed handling:

- `isConduitError`
- `isRateLimitError`
- `isRemoteFetchTimeoutError`
- `isTimeoutError`

## Matching

- Use `conduit.matching.create(...)` only when the user explicitly needs target-versus-group comparison.
- Keep `hiring_team_fit` intact when the user is doing the stable hiring comparison flow.

## Advanced primitives

- `primitives.media.upload(...)` for upload-once reuse flows and explicit speaker targeting with `mediaId`
- `primitives.media.get/list/delete/setRetentionLock(...)` for media management
- `primitives.jobs.get/cancel(...)` for job inspection and cancellation
- `primitives.entities.get/list/update(...)` for stable speaker entities
