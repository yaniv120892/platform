# Data

Load before adding a table, a migration, a Redis key or a storage bucket in any suite app.

## Postgres

Each app owns one Neon project; apps never read each other's database. Cross-app data moves only
through an app's API or MCP. Prisma is the ORM in every app, with a pooled URL for queries and an
unpooled URL for migrations (the names differ per app — see its `.env.example`).

Migrations run in the Vercel build (`prisma migrate deploy`). Investments migrates only on
production builds because its database variables are not scoped per environment; Parking gives
previews their own Neon `preview` branch. A new app gives Production and Preview separate Neon
branches, which is what makes migrating on every build safe.

Mastra-based chat (Expenses assistant, Investments advisor) keeps its memory in a separate `mastra`
schema in the same database, outside Prisma.

Rules Prisma cannot express — one owner per household, one active list or spot — are partial
unique indexes or CHECK constraints in migration SQL, mirrored in zod.

## Redis

Upstash over REST. Redis is a cache and a counter store: every read path works with it down, and
nothing lives only in Redis except short-lived codes, sessions (Expenses) and request state (the
extractor, 24 h). `app-kit`'s counter store is the client for rate limits and budgets.

When apps share an Upstash database, keys carry an app prefix so one app's keys cannot collide with
another's, plus an environment prefix outside production.

## Object storage

AWS S3, used by Expenses only: statement uploads under `imports/<userId>/`, transaction
attachments, backups. Access is by presigned URL to the browser and by service credentials to the
extractor.
