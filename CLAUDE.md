# CLAUDE.md

The cross-app architecture of the **My suite**: four household apps (Expenses, Investments,
Shopping, Parking) and the four shared packages they build on. Each app repo documents its own
internals; this repo holds what no single repo can answer — how the apps fit together, where each
piece runs, and the conventions every app follows.

Load it **before designing a feature** in any suite app, **before adding an app**, and **before
changing a shared package**. A feature that matches the suite conventions needs no decision here;
one that departs from them records the departure in [DECISIONS.md](DECISIONS.md) first.

## Docs

Short docs, one topic each. Load the one whose trigger matches your task.

- [DECISIONS.md](DECISIONS.md) — suite-wide decisions and why. Load before departing from a
  convention or adding one.
- [docs/system-map.md](docs/system-map.md) — every app, service and external dependency, and
  where it runs. Load before adding a service, a cron, a data store or an external API.
- [docs/shared-packages.md](docs/shared-packages.md) — what each package owns, the boundaries
  between them, the release flow. Load before writing code a sibling app could reuse, or changing
  a package.
- [docs/data.md](docs/data.md) — Postgres, Redis, object storage, migrations. Load before adding a
  table, a Redis key or a bucket.
- [docs/observability.md](docs/observability.md) — logs, errors, alerts, uptime, heartbeats,
  product events, and the parts configured outside code. Load before adding a route, a cron or a
  product event.
- [docs/households-and-agents.md](docs/households-and-agents.md) — the household model and the two
  agent channels every app exposes. Load before any household-scoped write or agent-facing feature.
- [docs/new-app.md](docs/new-app.md) — the checklist a new suite app follows. Load before starting
  a new app.

## Keeping it current

A PR in an app or package that changes a fact stated here — a URL, a store, a cron, a convention —
updates the doc here in the same change set. State the rule the suite now follows; git log holds
the story.
