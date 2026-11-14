# Decisions

One line per suite-wide decision that is hard to reverse, with the reason. An app's own decisions
live in its `DECISIONS.md`; the linked doc holds the detail.

| Topic | Decision | Why |
| --- | --- | --- |
| Suite | Expenses, Investments, Shopping and Parking read as one product: one design, one app switcher, one household model, one agent contract. → [docs/shared-packages.md](docs/shared-packages.md) | A household uses all four; each should feel like the same app. |
| Publishing | Free, agent-native, no payments; app and package repos are public. | Public repos keep GitHub Actions minutes free and let apps install packages without tokens. |
| Hosting | Vercel, one project per app, deployed by Git integration; Pro plan. → [docs/system-map.md](docs/system-map.md) | Hobby paused the apps on CPU in 2026-11; Pro removes that ceiling. |
| Node | Apps run Node 26 (`engines.node` `26.x`, which Vercel follows); the shared CI gate defaults to it; packages accept `>=22`. → [docs/shared-packages.md](docs/shared-packages.md) | The active LTS, and Vercel's default runtime: CI and production run the same major. |
| Region | Functions run in their database's region; new apps start in `fra1` with Neon and Upstash in Frankfurt. → [docs/system-map.md](docs/system-map.md) | Function, database and Redis in one region keep each request's round trips short; Binance refuses US callers. |
| Database | One Neon Postgres project per app via Prisma; apps never share a database. → [docs/data.md](docs/data.md) | An app can change its schema without coordinating; cross-app reads go through an API. |
| Redis | Upstash, as cache and counter store only. → [docs/data.md](docs/data.md) | Every read path keeps working when Redis is down. |
| Shared code | Four packages as git dependencies pinned by tag; a version bump in the same PR as the change. → [docs/shared-packages.md](docs/shared-packages.md) | No registry to run; Vercel caches git dependencies by version. |
| Households | Per-household data, one household per user, `OWNER`/`EDITOR`/`VIEWER`, role read from the database per request. → [docs/households-and-agents.md](docs/households-and-agents.md) | Family members share data; a role change takes effect at once. |
| Accounts | One account per person across the suite, owned by `my-accounts` (Better Auth). Apps sign in through it with OpenID Connect and keep their own session and database; it owns household membership and each member's role per app. → [docs/accounts.md](docs/accounts.md) | One sign-in flow means a fix lands once, and a family shares across apps with a different role in each. |
| Agents | Every app is reachable by the personal agent over MCP, and browser-only agents over `/agent` pages. → [docs/households-and-agents.md](docs/households-and-agents.md) | The suite is meant to be run by an agent as much as by a person. |
| Cost fences | Public routes rate-limited and fail open; paid work behind a daily budget and kill switch that fail closed. → [docs/shared-packages.md](docs/shared-packages.md) | Locking the owner out is worse than a gap in limits; an outage must never mean an unbounded bill. |
| Observability | Better Stack for logs, uptime and heartbeats; Sentry errors only; Telegram for ops alerts; PostHog US server-side only. → [docs/observability.md](docs/observability.md) | One place per signal across all apps, inside free or cheap tiers. |
| Docs | Each repo keeps `DECISIONS.md` plus one short doc per topic; no specs or plans committed. | A spec of shipped work drifts, and readers cannot tell it from intent. |
