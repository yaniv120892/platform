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

### Key prefix

Assume every Upstash database is shared: Shopping and Parking use one Marketplace store, and each
app's previews and local runs point at its production database. So every key starts with one
prefix, applied at the app's single Redis chokepoint, never by callers:

| `VERCEL_ENV` | Prefix |
| --- | --- |
| `production` | `<service>:` |
| `preview` | `<service>:preview:<branch>:` |
| anything else | `<service>:<VERCEL_ENV>:`, or `<service>:local:` when unset |

- `<service>` is the app's observability `service` name (`my-shopping`, `park-pal`). The extractor
  keeps `excel-extraction`, the namespace its production keys already use.
- `<branch>` is `VERCEL_GIT_COMMIT_REF`, falling back to `VERCEL_DEPLOYMENT_ID`, so two previews
  never share a key and a branch keeps its keys across pushes.
- Renaming a production key loses what it holds. A prefix change moves production keys only when
  every key is a TTL'd counter or cache; an app with state in Redis keeps its production keys.

Two apps hold state in production Redis and keep those keys bare until a dual-read migration
moves them: the Expenses website (sessions; its previews use `preview:<commit>:` for caches and
`preview:<branch>:` for sessions, without the `<service>:` segment) and Investments (login codes;
its previews follow the table). Verify against each app's Redis chokepoint.

The archived `my-expenses-app` repo's `docs/redis.md` describes an older `<service>:<environment>:`
format, `production` included. It is superseded by this section.

## Object storage

AWS S3, used by Expenses only: statement uploads under `imports/<userId>/`, transaction
attachments, backups. Access is by presigned URL to the browser and by service credentials to the
extractor.
