# Actovia skills.sh BFF

A small Hono relay used by [Actovia desktop](https://github.com/actovia-ai/actovia) and the [CLI](https://github.com/actovia-ai/cli). Requests under `/api/v1/*` are forwarded to the fixed `https://skills.sh` upstream with a fresh Vercel OIDC token, keeping that credential out of clients. The configured public relay is `https://skills-sh.yancey.app`.

## Behavior

`src/index.ts` exports the Hono app for Vercel. It preserves the upstream path, query, method, body, status and content type, and forwards rate-limit / Retry-After headers. If OIDC token acquisition fails, it returns `500 oidc_token_unavailable`. The root is a simple health/welcome response, not the skills API. This relay serves skill metadata and files; installation and local skill state belong to the desktop or CLI.

## Development

```bash
bun install
bun run dev   # Hono local development via Bun
```

Actual upstream requests require a valid Vercel OIDC environment. For the linked Vercel project, enable OIDC Federation and use the Vercel CLI to run with its development environment:

```bash
vercel link
vercel env pull .env.local
vercel dev
```

Tokens expire; refresh the local environment when needed. Never commit `.env.local`, OIDC tokens or `.vercel` project credentials.

## Deploy

```bash
vercel build
vercel deploy
```

`vercel deploy` creates a deployment; production promotion is a separate hosting decision. Verify `/api/v1/registry` against the deployment URL and inspect response status/rate-limit headers. Clients may override their relay URL with `ACTOVIA_SKILLS_BFF_URL`.
