# Rust reference

## Package identity

- Install: `cargo add conduit-rs`
- Import from crate: `use conduit_rs::...`
- Runtime target: server-side Rust

## First report

```rust
use conduit_rs::{
    Conduit, ReportCreate, ReportTemplate, Source, Target, WebhookEndpoint,
};

let conduit = Conduit::builder("sk_...")
    .max_retries(2)
    .build()?;

let media = conduit
    .primitives()
    .media
    .upload(Source::path("recordings/demo-call.wav"), None, None)
    .await?;

let receipt = conduit
    .reports()
    .create(
        ReportCreate::new(
            Source::media_id(media.media_id.clone()),
            ReportTemplate::GeneralReport,
            Target::hint("the interviewed person"),
        )
        .webhook(WebhookEndpoint::new("https://my-app.com/webhooks/conduit"))
        .idempotency_key("signup-call-42"),
    )
    .await?;

println!("uploaded {} and queued report job: {}", media.media_id, receipt.job_id);
```

## First webhook

```rust
use conduit_rs::{Conduit, WebhookEvent};
use http::HeaderMap;

async fn handle_webhook(
    conduit: &Conduit,
    body: &[u8],
    headers: &HeaderMap,
    secret: &str,
) -> Result<(), conduit_rs::Error> {
    conduit.webhooks().verify_signature(body, headers, secret)?;

    match conduit.webhooks().parse_event(body)? {
        WebhookEvent::ReportCompleted(event) => {
            if has_seen_event(&event.id) {
                return Ok(());
            }

            let report = conduit.reports().get(&event.report_id).await?;
            mark_seen_event(&event.id);
            persist_report(report);
        }
        _ => {}
    }

    Ok(())
}
```

## Framework fit

- If the repo uses `axum`, prefer body extractors that preserve raw bytes for verification.
- If the repo uses `actix-web` or `warp`, keep the exact raw body and headers before parsing JSON.
- If no framework is implied, keep examples on plain `&[u8]` plus `HeaderMap`.

## Stable surface

- `conduit.reports().create(...)`
- `conduit.reports().get(...)`
- `conduit.matching().create(...)`
- `conduit.matching().get(...)`
- `conduit.webhooks().verify_signature(...)`
- `conduit.webhooks().parse_event(...)`
- `conduit.primitives().media(...)`
- `conduit.primitives().jobs(...)`
- `conduit.primitives().entities(...)`

## Webhooks

Use this order:

1. Read the exact raw request body.
2. `conduit.webhooks().verify_signature(body, headers, secret)?`
3. `match conduit.webhooks().parse_event(body)? { ... }`
4. Deduplicate on `event.id` before side effects.
5. Handle typed variants like `WebhookEvent::ReportCompleted` first.
6. Fetch the final resource after completion.

Signature verification expects the `conduit-signature` header.

## Core request types

- `Source` for input media
- `Target` for selected speaker in reports
- `SubjectRef` for matching subjects
- `WebhookEndpoint` for completion callbacks
- `WaitOptions` and `StreamOptions` for polling helpers

## Runtime notes

- `Source::file(...)`, `Source::url(...)`, `Source::path(...)`, `wait()`, `stream()`, and webhook verification are supported in server-side Rust.
- `Source::url(...)` performs an SDK-side fetch and then uploads the fetched bytes to Conduit.
- `wait()` and `stream()` are local-dev or tooling helpers, not the default production path.

## Source shapes

Use upload plus `Source::media_id(...)` as the default onboarding path.

- `Source::media_id("med_...")`
- `Source::file("call.wav", bytes)`
- `Source::url("https://example.com/call.wav")`
- `Source::path("recordings/demo-call.wav")`

## Target selection

Use `Target::hint("the interviewed person")` first when the user needs one person selected from uploaded media.

Other stable selectors:

- `Target::dominant()`
- `Target::entity("ent_123")`
- time-range targeting when the user has explicit bounds

## Local wait or stream

```rust
use std::time::Duration;

let report = receipt.handle.wait_for(Duration::from_secs(300)).await?;
```

## Matching

- Use `conduit.matching().create(...)` only for explicit target-versus-group comparison.

## Advanced primitives

- Prefer `Conduit::builder(...)` examples over ad hoc client setup.
- Keep examples async and `tokio`-compatible.
- Use `conduit_rs::Error` as the SDK error type in examples.
- Use primitives only when the user needs lower-level media, jobs, or entity workflows, including upload-once reuse before report creation.
