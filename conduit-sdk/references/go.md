# Go reference

## Package identity

- Install: `go get github.com/mappa-ai/conduit-go`
- Import: `conduit "github.com/mappa-ai/conduit-go"`

## First report

```go
client, err := conduit.New(os.Getenv("CONDUIT_API_KEY"))
if err != nil {
	log.Fatal(err)
}
defer client.Close()

media, err := client.Primitives.Media.Upload(context.Background(), conduit.MediaUploadRequest{
	Source: conduit.SourcePath("recordings/demo-call.wav"),
})
if err != nil {
	log.Fatal(err)
}

receipt, err := client.Reports.Create(context.Background(), conduit.CreateReportRequest{
	IdempotencyKey: "signup-call-42",
	Output: conduit.ReportOutputRequest{Template: conduit.ReportTemplateGeneralReport},
	Source: conduit.SourceMediaID(media.MediaID),
	Target: conduit.TargetMagicHint("the interviewed person"),
	Webhook: &conduit.Webhook{URL: "https://my-app.com/webhooks/conduit"},
})
if err != nil {
	log.Fatal(err)
}

log.Printf("uploaded %s and accepted report job %s with status %s", media.MediaID, receipt.JobID, receipt.Status)
```

## First webhook

```go
func handleConduitWebhook(client *conduit.Client) http.HandlerFunc {
	secret := os.Getenv("CONDUIT_WEBHOOK_SECRET")

	return func(w http.ResponseWriter, r *http.Request) {
		defer r.Body.Close()

		payload, err := io.ReadAll(r.Body)
		if err != nil {
			http.Error(w, "failed to read body", http.StatusBadRequest)
			return
		}

		if err := client.Webhooks.VerifySignature(payload, r.Header, secret, 5*time.Minute); err != nil {
			http.Error(w, "invalid signature", http.StatusUnauthorized)
			return
		}

		event, err := client.Webhooks.ParseEvent(payload)
		if err != nil {
			http.Error(w, "invalid event", http.StatusBadRequest)
			return
		}

		if hasSeenEvent(event.ID) {
			w.WriteHeader(http.StatusOK)
			return
		}

		if event.Type == "report.completed" {
			data := event.Data.(map[string]any)
			reportID := data["reportId"].(string)
			report, err := client.Reports.Get(r.Context(), reportID)
			if err != nil {
				http.Error(w, "failed to fetch report", http.StatusBadGateway)
				return
			}
			markSeenEvent(event.ID)
			_ = report
		}

		w.WriteHeader(http.StatusOK)
	}
}
```

## Framework fit

- If the repo implies `net/http`, use `http.HandlerFunc` examples.
- If the repo uses Gin, Echo, or Chi, preserve the exact raw body bytes before binding JSON.
- If no framework is implied, stay on standard library handler shapes.

## Stable surface

- `client.Reports.Create(...)`
- `client.Reports.Get(...)`
- `client.Matching.Create(...)`
- `client.Matching.Get(...)`
- `client.Webhooks.VerifySignature(...)`
- `client.Webhooks.ParseEvent(...)`
- `client.Primitives.Entities.*`
- `client.Primitives.Media.*`
- `client.Primitives.Jobs.*`

## Webhooks

Use this order:

1. `payload, err := io.ReadAll(r.Body)`
2. `client.Webhooks.VerifySignature(payload, r.Header, secret, 5*time.Minute)`
3. `event, err := client.Webhooks.ParseEvent(payload)`
4. Deduplicate on `event.ID`
5. Handle `report.completed` first
6. Fetch the final report or matching object after `*.completed`

## Source helpers

Use upload plus `SourceMediaID(...)` as the default onboarding path. The canonical source helpers are:

- `conduit.SourceMediaID("med_...")`
- `conduit.SourceBytes("call.wav", audioBytes)`
- `conduit.SourceURL("https://example.com/call.wav")`
- `conduit.SourcePath("./call.wav")`
- `conduit.SourceReader("call.wav", reader)`

Use `Source.WithLabel(...)` when the caller wants a custom media label.

## Target selection

Use `conduit.TargetMagicHint("the interviewed person")` first when the user needs one person selected from uploaded media.

Other stable selectors:

- `conduit.TargetDominant()`
- `conduit.TargetEntityID("ent_123")`
- `conduit.TargetTimeRange(startSeconds, endSeconds)`

## Local wait or stream

- `receipt.Handle.Wait(...)` for local tooling
- `receipt.Handle.Stream(...)` for live SSE job events
- `StreamOptions.LastEventID` to resume explicit stream reconnects

## Matching

- Use `client.Matching.Create(...)` only for explicit target-versus-group comparison.
- Keep `conduit.MatchingContextHiringTeamFit` intact for the stable hiring flow.

## Advanced primitives

- Pass `context.Context` through every SDK call.
- Use `client.Primitives.Media.Upload(...)` when the user wants reusable uploaded media or explicit speaker targeting from uploaded media.
- Use `client.Primitives.Jobs.*` only for job inspection or cancellation.
- Use `client.Primitives.Entities.*` only for stable entity management.
- Call out `client.Close()` in examples that own the client lifecycle.
