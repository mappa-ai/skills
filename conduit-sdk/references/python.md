# Python reference

## Package identity

- Install: `pip install mappa-conduit`
- Import: `from conduit import Conduit`
- Runtime: Python 3.12+

## First report

```python
from conduit import Conduit

conduit = Conduit(api_key="sk_...")

media = conduit.primitives.media.upload(path="recordings/demo-call.wav")

receipt = conduit.reports.create(
    source={"media_id": media.media_id},
    output={"template": "general_report"},
    target={"strategy": "magic_hint", "hint": "the interviewed person"},
    webhook={"url": "https://my-app.com/webhooks/conduit"},
    idempotency_key="signup-call-42",
)

print(media.media_id, receipt.job_id, receipt.status)
```

## First webhook

```python
from conduit import Conduit

conduit = Conduit(api_key="sk_...")


def handle_webhook(raw_payload: bytes, headers: dict[str, str]) -> None:
    conduit.webhooks.verify_signature(raw_payload, headers, secret="whsec_...")
    event = conduit.webhooks.parse_event(raw_payload)

    if has_seen_event(event.id):
        return

    if event.type == "report.completed":
        report = conduit.reports.get(event.data["reportId"])
        mark_seen_event(event.id)
        persist_report(report)
```

## Framework fit

- If the repo uses FastAPI, prefer `raw_payload = await request.body()`.
- If the repo uses Flask or Django, preserve the exact raw request body bytes before parsing JSON.
- If the repo does not imply a framework, keep examples on plain `bytes` plus header dicts.

## Stable surface

- `conduit.reports.create(...)`
- `conduit.reports.get(...)`
- `conduit.matching.create(...)`
- `conduit.matching.get(...)`
- `conduit.webhooks.verify_signature(...)`
- `conduit.webhooks.parse_event(...)`
- `conduit.primitives.entities.*`
- `conduit.primitives.media.*`
- `conduit.primitives.jobs.*`

## Webhooks

Use this order:

1. Keep the exact raw request body as `bytes`.
2. Call `conduit.webhooks.verify_signature(raw_payload, headers, secret=...)`.
3. Call `event = conduit.webhooks.parse_event(raw_payload)`.
4. Deduplicate on `event.id`.
5. Handle `report.completed` first.
6. Fetch the final report or matching object on `*.completed`.

Header lookup is case-insensitive and uses `conduit-signature`.

## Common source forms

Use upload plus `media_id` as the default onboarding path.

- `source={"path": "recordings/demo-call.wav"}`
- `source={"url": "https://storage.example.com/call.wav"}`
- `source={"media_id": media.media_id}` after `conduit.primitives.media.upload(...)`

## Target selection

Use `magic_hint` first when the user needs one person selected from uploaded media.

```python
{"strategy": "magic_hint", "hint": "the interviewed person"}
```

Other stable selectors:

- `{"strategy": "dominant"}`
- `{"strategy": "entity_id", "entity_id": "ent_123"}`
- `{"strategy": "timerange", "time_range": {"start_seconds": 15, "end_seconds": 45}}`

## Local wait or stream

- `receipt.handle.wait(timeout_ms=...)` for local scripts
- `receipt.handle.stream(timeout_ms=...)` for live job events
- `receipt.handle.job()` for current job state
- `receipt.handle.cancel()` for cancellation
- `receipt.handle.report()` when the caller wants the completed report only if it already exists

## Matching

Use `conduit.matching.create(...)` only for explicit target-versus-group comparisons and keep the group values as entity or media-backed subjects.

## Advanced primitives

- Use `primitives.media.upload(...)` when the same recording will be reused or when speaker targeting should happen from uploaded media.
- Use `primitives.jobs.*` only for job inspection or cancellation.
- Use `primitives.entities.*` only for stable entity management.
