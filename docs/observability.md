# Observability

Load before adding a route, a cron, a background job or a product event, or when an alert fires.

Every app wires this once through `@yaniv120892/observability` with its own `service` name. The
package README is the API; this doc is what the suite expects of each app.

## Per app

- **Logs**: pino to stdout; warn and above, plus records marked `ship: true`, go to Better Stack.
  Vercel keeps runtime logs only briefly, so anything worth reading later is warn+ or `ship: true`.
- **Errors**: Sentry, errors only — no tracing, no replay. Source maps upload when the three
  `SENTRY_*` build variables are set.
- **Ops alerts**: Telegram, one shared bot and ops chat, prefixed with the service name, capped
  per alert type per hour. Every unhandled 5xx alerts; a failed cron or a reached global budget
  alerts by name.
- **Health**: `/api/health` is liveness and touches nothing; `/api/health/deep` probes the
  database (and Redis where it is a dependency), rate-limited and cached.
- **Heartbeats**: each cron pings its own Better Stack heartbeat after a successful run.
- **Product events**: PostHog US, server-side only, sent from production only. Events are named
  `object_verb` in the app's `productEvents` module, carry a `channel` (web, agent) wherever an agent
  can cause them, and identify people by internal id only.

## Configured outside code

These live in dashboards, not repos. Change them by hand and update this list.

| Where | What |
| --- | --- |
| Better Stack | one HTTP log source per app; per app a shallow monitor on `/api/health` every 3 min and a deep one on `/api/health/deep` every 60 min; one heartbeat per cron; missed beats and failed monitors email the owner |
| Sentry | one project per app |
| PostHog | one shared project (692842, US) for every app |
| Telegram | the ops bot and alert chat; the Expenses user bot; the Investments summary chat |
| Vercel | Pro plan; project env vars; Marketplace integrations (Upstash KV in Shopping and Parking) |

## Weekly report

A GitHub Actions cron in the `observability` repo sends a Sunday Telegram report built from
PostHog traffic, and pings its own heartbeat.
