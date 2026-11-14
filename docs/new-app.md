# New suite app

Load before starting a new app in the suite. Copy from the newest app that has the piece
(Parking was built from Shopping) rather than re-deriving it.

1. **Repo**: public under `yaniv120892`; `CLAUDE.md`, `DECISIONS.md` and one short doc per topic
   under `docs/`. Next.js App Router, TypeScript, npm, `engines.node` `26.x`.
2. **Shared packages**: all four, pinned by tag → [shared-packages.md](shared-packages.md).
   `AppShell` with a new `APP_IDS` entry in `my-ui`, and the app's URL added to every sibling's
   app switcher.
3. **CI**: `dev-config`'s `node-ci.yml` gate, plus an integration job on a Postgres service.
4. **Vercel**: a project in `yaniv120892s-projects`, region `fra1` (the database's region), `buildCommand` that migrates
   then builds.
5. **Data**: a Neon project in Frankfurt with separate Production and Preview branches; Upstash
   for counters → [data.md](data.md).
6. **Auth**: passwordless (Google PKCE or emailed link/code), JWT in an `auth-token` cookie,
   `tokenVersion` for sign-out everywhere.
7. **Households and agents**: the household model, MCP and `/agent` pages →
   [households-and-agents.md](households-and-agents.md).
8. **Cost fences**: `app-kit` rate limits on public routes (fail open) and daily budgets with a
   kill switch on paid work (fail closed).
9. **Observability**: `createNextObservability`, health routes, Better Stack source, monitors and
   heartbeats, a Sentry project, product events → [observability.md](observability.md).
10. **This repo**: add the app to [system-map.md](system-map.md) and the dashboards list in
    [observability.md](observability.md).
