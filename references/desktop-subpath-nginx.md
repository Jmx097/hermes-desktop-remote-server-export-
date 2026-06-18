# Desktop-only nginx subpath for Hermes Desktop

Use this pattern when:

- the main public hostname must stay unchanged
- browser/root access remains behind existing nginx auth
- Hermes Desktop says `Could not reach gateway URL`
- you want the smallest reversible change that exposes only a dedicated dashboard base path

## Why it works

Hermes Desktop probes the configured base URL by requesting the dashboard backend's public `/api/status` on that same base URL. If nginx `auth_basic` blocks the root dashboard path, Desktop fails before it can negotiate token or OAuth auth.

A dedicated unauthenticated subpath such as `/desktop` gives Desktop a clean dashboard base URL while preserving the root host behavior for browsers.

## Example nginx shape

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

## Critical detail

Use a trailing slash on both:

- `location /desktop/`
- `proxy_pass http://127.0.0.1:9119/;`

That causes nginx to strip the `/desktop/` prefix before forwarding, so Hermes receives canonical backend paths like:

- `/api/status`
- `/api/ws`
- `/assets/...`

Without the correct slash pairing, the prefix handling can be wrong and Desktop probing/websocket upgrades may still fail.

## Verification

After reloading nginx, verify from outside the box:

- `https://host/desktop/api/status` returns 200
- Hermes Desktop remote URL should be the base subpath itself: `https://host/desktop`

If `/api/status` succeeds and shows:

- `"auth_required": false`
- `"auth_providers": []`

then Desktop should now reach the gateway but will still use token-mode auth rather than hosted sign-in.
