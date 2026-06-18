---
name: hermes-desktop-remote-server-export
description: Use when packaging a reusable, shareable Hermes setup for exposing Hermes Desktop remote access behind a server, reverse proxy, or subpath, including exportable skill artifacts and verification steps.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes, desktop, remote-access, server, reverse-proxy, export, packaging, nginx]
    related_skills: [hermes-agent, hermes-desktop-remote-access, hermes-remote-access, hermes-integrations-and-bridges]
---

# Hermes Desktop Remote Server Export

## Overview

Use this skill to package a reusable, shareable workflow for spinning up Hermes Desktop remote-access servers that other Hermes users can import directly. The goal is not just to explain remote access, but to hand someone a skill export that teaches them how to stand up the server safely, keep the dashboard/API split straight, expose the correct public URL, and verify the result end-to-end.

This is specifically for the **Hermes Desktop remote gateway/server** class of setup: a Hermes instance running on a VPS or remote host, optionally fronted by nginx/Caddy/Cloudflare, with a public URL that Hermes Desktop can use.

## When to Use

Use this skill when the user wants to:

- spin up a remote Hermes server for Hermes Desktop
- package the setup as a reusable skill they can share with other Hermes users
- expose Hermes dashboard/API over a reverse proxy
- preserve one hostname while separating dashboard and `/v1` API behavior
- troubleshoot session-token vs Sign in behavior in Hermes Desktop
- hand off a reproducible remote-server recipe rather than ad hoc setup notes

Do **not** use this skill for:

- generic OpenAI-compatible API hosting unrelated to Hermes Desktop
- local-only Hermes CLI setup with no public ingress
- realtime voice sidecars or telephony bridges unless they are explicitly part of the server packaging task

## Core model

Hermes remote deployments are easiest to reason about when you keep three layers separate:

1. **Hermes dashboard backend**
   - browser-facing/dashboard surface
   - commonly local `127.0.0.1:9119`
   - Hermes Desktop Remote gateway talks to this surface

2. **Hermes OpenAI-compatible API server**
   - machine/API surface under `/v1`
   - commonly local `127.0.0.1:8642`
   - for API clients, not the same thing as Desktop remote gateway

3. **Public reverse-proxy layer**
   - nginx/Caddy/Cloudflare/Tailscale/etc.
   - decides which paths are reachable and whether Desktop can hit `/api/status`

If a deployment gets weird, assume one of these layers is being confused with another.

## What the packaged export should include

A good shareable export should teach the recipient:

- which Hermes surface Hermes Desktop expects
- what public URL to paste into Hermes Desktop
- how `/` differs from `/v1`
- how to preserve an existing hostname without breaking browser auth
- how to diagnose session-token prompts
- how to verify the deployment before declaring success

The export should be self-contained enough that another Hermes user can install the skill and follow it without needing your original conversation.

## Recommended packaging pattern

When creating a shareable skill export for other Hermes users:

1. **Keep the main SKILL.md class-level and durable**
   - concepts
   - decision rules
   - verification workflow
   - common pitfalls

2. **Put concrete snippets in support files when needed**
   - nginx examples
   - checklists
   - upgrade notes
   - migration notes

3. **Optimize for importability**
   - the recipient should be able to install the skill by file/URL and use it immediately
   - avoid references to one-off hostnames, secrets, or your personal machine

4. **Write for operators, not end users**
   - the audience is someone running Hermes, not a random desktop app user

## Default operator workflow

### 1. Identify the intended public surface

Ask and verify:

- what hostname or tunnel URL will be public
- whether the user wants one hostname for both dashboard and API
- whether root `/` is already protected by basic auth or another layer
- whether Hermes Desktop must work against the root or can use a subpath like `/desktop`

### 2. Separate dashboard vs API server early

Keep this mapping explicit:

- dashboard/backend base URL: `https://host/` or `https://host/desktop`
- API base URL: `https://host/v1`

Rules:

- Hermes Desktop **Remote gateway** uses the dashboard/backend base URL
- OpenAI-compatible clients use `/v1`
- `/v1` being healthy does **not** prove Desktop remote mode is healthy

### 3. Verify the local upstreams

Check whether the deployment actually has both surfaces and where they bind:

- dashboard service, often `127.0.0.1:9119`
- API server, often `127.0.0.1:8642`

Do not assume defaults are live; verify the actual listeners and service config.

### 4. Shape the reverse proxy safely

Preferred shape when sharing one hostname:

- `/` → dashboard
- `/health` → API server health
- `/v1/` → API server

If root `/` is protected in a way that blocks Hermes Desktop probing, use a dedicated Desktop-safe subpath:

- `/desktop` → dashboard backend with auth disabled at the proxy layer for that subpath only

Critical nginx detail:

- `location /desktop/`
- `proxy_pass http://127.0.0.1:9119/;`

Both should end with `/` so the prefix is stripped and Hermes receives canonical backend paths like `/api/status` and `/api/ws`.

### 5. Interpret Desktop auth symptoms correctly

- **Sign in** usually means the backend is advertising a hosted auth provider
- **session token prompt** usually means token-mode dashboard auth or version skew
- **Could not reach gateway URL** usually means reverse-proxy or public `/api/status` reachability trouble before auth even begins

### 6. Verify end-to-end before packaging as done

Minimum verification:

- public dashboard path reachable
- public `/api/status` reachable on the exact Desktop base URL
- `/v1/models` works separately if API access matters
- Desktop can reach the gateway using the intended URL
- if applicable, confirm whether the result is token-mode or Sign in mode

## Exportable nginx pattern

For environments where the main root stays protected but Hermes Desktop needs a reachable public dashboard base URL, package this pattern in your guidance:

```nginx
location = /desktop {
    auth_basic off;
    return 301 https://$host/desktop/;
}

location /desktop/ {
    auth_basic off;
    proxy_pass http://127.0.0.1:9119/;
    proxy_http_version 1.1;
    proxy_set_header Host 127.0.0.1:9119;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Origin http://127.0.0.1:9119;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_read_timeout 86400;
}
```

Then tell the recipient to verify:

- `https://host/desktop/api/status`
- Hermes Desktop remote URL: `https://host/desktop`

## Version-skew rule

When the docs and runtime disagree, treat **backend version skew** as a primary hypothesis.

Typical pattern:

- hostname and TLS are fine
- `/v1/*` works
- dashboard loads
- Hermes Desktop asks for a session token instead of showing Sign in
- docs describe a newer sign-in flow than the runtime actually supports

User-facing interpretation:

- the hostname may already be fine
- the runtime may simply be older than the docs
- a raw session token can be a temporary unblocker
- the durable fix is upgrading Hermes and re-checking Desktop behavior

## Common Pitfalls

1. **Telling users to put `/v1` into Hermes Desktop Remote gateway.**
   Wrong surface.

2. **Declaring success because `/v1/models` works.**
   That only proves the API server path.

3. **Breaking a live browser/dashboard route during troubleshooting.**
   Prefer read-only checks and additive subpaths first.

4. **Putting host-specific values into a shared export.**
   Keep the exported skill generic and reusable.

5. **Assuming session-token mode means total failure.**
   Sometimes it means the backend is older or still in token-style auth.

6. **Forgetting websocket/path-prefix behavior on `/desktop`.**
   Wrong slash pairing on nginx can make probing or `/api/ws` fail.

## Verification Checklist

- [ ] The recipient can clearly distinguish dashboard base URL from `/v1` API base URL
- [ ] The skill states that Hermes Desktop Remote gateway targets the dashboard/backend surface
- [ ] Reverse-proxy examples preserve canonical backend paths
- [ ] The skill includes a dedicated `/desktop` fallback pattern for proxy-auth conflicts
- [ ] The skill explains session-token vs Sign in behavior
- [ ] The skill warns about version skew between docs and runtime
- [ ] The packaged instructions include real verification probes, not just config snippets
- [ ] No hostnames, secrets, or one-off environment details leak into the export

## Suggested share workflow

If the goal is to hand this to other Hermes users, the cleanest distribution options are:

1. keep it as a local skill and export/share the SKILL.md directory
2. publish the SKILL.md in a repo or gist and have users install it by URL
3. bundle support references alongside the skill if the examples are long

For a reusable export, prefer a stable raw URL or repo path over pasting fragments into chat.

## References

- `hermes-desktop-remote-access` — deeper Desktop symptom interpretation
- `hermes-remote-access` — dashboard vs API split and no-downtime proxy guidance
- `hermes-integrations-and-bridges` — broader integration packaging pattern
