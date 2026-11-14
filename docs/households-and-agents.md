# Households and agents

Load before any household-scoped query or write, a role check, an invite flow, or a feature an
agent should be able to use.

## Households

Every suite app stores data per household, with the same model:

- A user belongs to exactly one household, as `OWNER`, `EDITOR` or `VIEWER`. Each household has
  exactly one owner; ownership is not transferable.
- The actor's household and role are read from the database on every request, never from the JWT,
  so a removed member or changed role takes effect on the next request.
- Every write of household data goes through one write path that checks the role and writes the
  `HouseholdActivity` row in the same transaction.
- Invites are short single-use codes at a role the owner picks (never `OWNER`), accepted by an
  explicit POST to `/join`. Sign-in never joins a household.

The suite is moving to one household across apps with a role per app, owned by `my-accounts`:
[accounts.md](accounts.md). An app follows the rules above until it moves.

A new household-scoped feature reuses the app's write path and activity log; it adds a row cap per
household for anything a member can create in bulk.

## Agent channels

The owner's personal agent reaches each app two ways, and every feature a user can do should be
reachable by both:

- **MCP** at `POST /api/mcp`: Streamable HTTP, stateless, JSON responses; GET and DELETE answer
  405. Authenticated by a per-user bearer token stored only as a SHA-256 hash;
  `app-kit/token-format` is the shared layout. A viewer's token is refused every write, by the same
  guard the browser hits. Rate-limited per client address.
- **`/agent` pages** for browser-only agents: server-rendered, no client JavaScript, form posts
  answering 303, authenticated by an `agent-session` cookie scoped to `/agent`. Shopping and Parking
  have them; Expenses and Investments have MCP only.

An agent acts as its user, in that user's current household and role. Services take a request
actor and never branch on the channel; product events record the channel instead.

Settings shows a paste-ready briefing per token. The briefing lists the registered tools; a test
asserts the two match.
