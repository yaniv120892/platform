# Accounts

Load before changing sign-in in any app, before adding an app's sign-in, or before changing who
belongs to a household or what role they hold in an app.

## The rule

- A person has one account for the whole suite. `my-accounts` owns it: email, Google link, and the
  sign-in methods, which are Google and the emailed link and code only.
- An app signs a person in through `my-accounts` with OpenID Connect: authorization code with
  PKCE, as a trusted client with no consent screen. The app then sets its own `auth-token`
  session cookie and finds its local user by account id. Each app keeps its own database.
- Each app is opt-in per account: a new account has only the app it signed up in.
- One household holds the family. `my-accounts` owns its members and each member's role per app
  (`OWNER`, `EDITOR`, `VIEWER` or none), with one Owner per app, and pushes an app its roles with a signed server-to-server
  call. The app still reads the role from its own database on every request.
- A member with no role in an app chooses to ask its Owner for access or to start their own data
  there. Any member can start an app for the family and becomes its Owner in that app.
- MCP tokens and `/agent` sessions stay in each app, tied to the local user.

## Apps that sign in on their own

An app that does not sign in through `my-accounts` signs people in with its own flow, documented in
that app's repo, and follows the one-role household in
[households-and-agents.md](households-and-agents.md) — load it before a household query or role
check.
