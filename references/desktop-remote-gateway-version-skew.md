# Hermes Desktop remote gateway: version-skew pattern

## Durable lesson

When Hermes Desktop troubleshooting contradicts the official docs, check for **backend version skew** before changing reverse proxy, auth, or API settings.

## Pattern

Observed class of failure:

- public hostname is valid and TLS works
- `/v1/*` works with bearer/API key
- dashboard is reachable behind `/`
- Hermes Desktop asks for a **session token** instead of showing **Sign in**
- current docs describe a provider-advertised sign-in flow

Most likely explanation:

- the running Hermes dashboard backend is older than the docs and does not yet implement the newer advertised auth-provider behavior

## Interpretation rules

1. Hermes Desktop Remote gateway means the **dashboard backend**, not `/v1`.
2. `/v1` success proves only that the API server path works.
3. A session-token prompt is a backend-auth clue, not an API-key clue.
4. If the docs mention auth-provider fields/behavior that the runtime lacks, treat that as version mismatch, not operator error.

## Good operator sequence

1. Confirm proxy routing for `/` vs `/v1/*`.
2. Compare installed Hermes version to upstream/docs recency.
3. Inspect whether the dashboard still emits an in-page session token.
4. Use that token only as a temporary unblocker.
5. Upgrade Hermes for the durable fix.

## Temporary workaround

Older dashboards may expose an ephemeral session token in the rendered HTML/JS bootstrap.

Use only to unblock access temporarily:

- token changes on dashboard restart
- token is not the intended steady-state remote-auth workflow

## User-facing explanation that tends to land well

"The hostname is fine. The backend is older than the docs you’re following, so Desktop is falling back to a raw session-token flow. We can use that token right now, but the real fix is upgrading Hermes so the backend advertises the proper sign-in provider."
