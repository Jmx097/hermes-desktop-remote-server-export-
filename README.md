# Hermes Desktop Remote Server Export

A reusable Hermes skill for packaging and operating remote Hermes Desktop server deployments.

## Install

```bash
hermes skills install https://raw.githubusercontent.com/<owner>/<repo>/<branch>/hermes-desktop-remote-server-export/SKILL.md
```

## What it covers

- Hermes Desktop remote gateway server setup
- dashboard vs `/v1` API split
- reverse-proxy guidance
- `/desktop` nginx subpath fallback
- session-token vs Sign in troubleshooting
- version-skew diagnosis
- shareable operator workflow for other Hermes users

## Files

```text
hermes-desktop-remote-server-export/
├── SKILL.md
└── references/
    ├── desktop-remote-gateway-version-skew.md
    ├── desktop-subpath-nginx.md
    └── public-sharing.md
```

## Notes

This is meant to be generic and reusable. Replace placeholder repo URLs with your own hosted raw files before sharing.
