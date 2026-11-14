# Shared packages

Load before writing code a sibling app could reuse, before changing a package, or before bumping a
package in an app.

All four are public repos under `yaniv120892`, installed as git dependencies. Each README is the
reference for its API; this doc holds what goes where.

| Package | Owns | Reach for it when |
| --- | --- | --- |
| `my-ui` | MUI theme, `AppShell` and app switcher, he/en i18n with RTL, locale-aware formatting | a page, a component, a string, a number or date on screen |
| `app-kit` | server guards: Upstash counter store, rate limits, daily budgets with kill switches, security headers, safe redirects, secret and cron checks, signed token format | a route a stranger can drive, paid work (LLM, mail, geocoding), a token, a cron route |
| `observability` | pino logging to Better Stack, Sentry, Telegram ops alerts, health routes, heartbeats, PostHog events, weekly report | a log line, an error report, an alert, a health route, a product event |
| `dev-config` | ESLint, Prettier, tsconfig, lint-staged presets and the reusable `node-ci.yml` gate (Node 26 by default) | lint, format, typecheck or CI setup |

## Where new shared code goes

Code lands in a package once a second app needs it. Until then it lives in the app, written so it
can move: no app imports, inputs passed in. The first app to build a feature is the template the
next app copies from (Shopping is the template for auth, agent channels, the shell and households).

The packages stay independent: `app-kit` takes its logger and alerter as parameters and accepts
`observability`'s object by shape. A package that needs another's behaviour takes it as a
parameter.

## Release and pin

1. Bump `version` in the same PR as the change — Vercel caches git dependencies by version, so a
   new commit at an old version never reaches a deploy.
2. Squash-merge, then `gh release create v<version> --generate-notes`.
3. Apps pin the tag (`github:yaniv120892/<pkg>#vX.Y.Z`), and for `dev-config` also the CI
   `uses: …/node-ci.yml@vX.Y.Z` line. Check the lockfile `resolved` SHA matches the tag.

A breaking default change gets an "Upgrading to" section in the package README, and the bump PR in
each app follows it.
