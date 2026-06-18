# Share this skill publicly

This skill is designed to be shared with other Hermes users.

## Option 1: Host the raw files in GitHub

Recommended layout:

```text
hermes-desktop-remote-server-export/
├── SKILL.md
└── references/
    ├── desktop-subpath-nginx.md
    └── desktop-remote-gateway-version-skew.md
```

Then users can install from the raw SKILL.md URL:

```bash
hermes skills install https://raw.githubusercontent.com/<owner>/<repo>/<branch>/hermes-desktop-remote-server-export/SKILL.md
```

## Option 2: Publish in a skills repo

If you maintain a shared Hermes skills repo, place the folder under an appropriate category path and point users to the raw `SKILL.md` URL.

## What to keep out of the public export

Do not include:

- private hostnames
- API keys or tokens
- user-specific paths unless they are clearly examples
- environment-specific ports unless they are framed as common defaults

## Suggested README copy

```md
# Hermes Desktop Remote Server Export

A reusable Hermes skill for packaging and operating remote Hermes Desktop server deployments.

Install:

```bash
hermes skills install https://raw.githubusercontent.com/<owner>/<repo>/<branch>/hermes-desktop-remote-server-export/SKILL.md
```

Use when you need to expose Hermes Desktop remote access behind nginx or another reverse proxy, keep dashboard and `/v1` API surfaces straight, and troubleshoot session-token vs Sign in behavior.
```
