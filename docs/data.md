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
prefix, added where the app builds its Redis client or counter store, never by callers:

| `VERCEL_ENV` | Prefix |
| --- | --- |
| `production` | `<service>:` |
| `preview` | `<service>:preview:<branch>:` |
| anything else | `<service>:<VERCEL_ENV>:`, or `<service>:local:` when unset |

- `<service>` is the app's observability `service` name (`my-shopping`, `park-pal`). The extractor
  keeps `excel-extraction`, the namespace its production keys already use.
- `<branch>` is `VERCEL_GIT_COMMIT_REF`, falling back to `VERCEL_DEPLOYMENT_ID`, then `unknown`. Two
  branches never share keys, and a branch keeps its keys across pushes.
- Key names after the prefix start with the key's own domain term (`rate_limit:`, `session:`), so a
  production key cannot read as a preview's.
- Renaming a production key loses what it holds, so a prefix change moves production keys only when
  every key is a TTL'd counter or cache. An app whose production keys hold state (sessions, login
  codes, request state) keeps them until a dual-read migration moves them, and until then its
  production keys may stay bare only in an Upstash database no other app uses. Its own Redis doc
  names the exception; the Expenses website and Investments are such apps today.
- Keys a prefix change leaves behind expire by their TTL.
- A preview counts against its own rate limits and daily budgets, not production's, so a limit an
  upstream sets per account, such as a geocoder's one request a second, can be exceeded when a
  preview and production call it together.

The archived `my-expenses-app` repo's `docs/redis.md` describes an older `<service>:<environment>:`
format, `production` included. It is superseded by this section.

## Object storage

AWS S3, used by Expenses only: statement uploads under `imports/<userId>/`, transaction
attachments, backups. Access is by presigned URL to the browser and by service credentials to the
extractor.
