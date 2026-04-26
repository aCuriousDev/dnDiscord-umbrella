# Deployment

Infrastructure overview for the DnDiscord POC. Covers where things run, how deploys happen, and what env vars are required.

---

## Environments

| Environment | URL | Host | Branch tracked |
|---|---|---|---|
| App (back + front) | <https://dndiscord.cadran.app/> | Dokploy | `dev` |
| Landing | <https://dndiscord-landing.cadran.app/> | Dokploy | `dev` |

Both `epi-esp-back` and `epi-esp-front` auto-deploy from the `dev` branch - this is the current POC convention and may change as the project matures. Releases tagged from `main` (`v0.1.x`) are for delivery/handoff, not the auto-deploy trigger.

---

## Deploy flow

1. Push (or merge a PR) to `dev` on either repo.
2. CI runs - back: `back/.github/workflows/ci.yml`; front: `front/.github/workflows/ci.yml`. Both pipelines must pass before the branch is considered healthy.
3. Dokploy detects the push via GitHub App webhook, rebuilds the Docker image, and swaps the container behind the production reverse proxy.
4. Back: EF migrations run at startup, not during the build. If a migration fails, the new container never becomes healthy and the previous one keeps serving traffic (`/api/health` is the health check).

---

## Containers

- **back** - built from `back/Services.Dockerfile` (multi-stage .NET 9 build). Runs the `DnDiscordAPI` ASP.NET Core process on port `5054`. Depends on PostgreSQL 16.
- **front** - built from `front/dndiscord-esp/Dockerfile` (Node 20 build stage → `nginx:alpine` runtime). Serves the Vite SPA on port `80`. `VITE_API_URL` is baked in at image build time via a Docker build arg.
- **landing** - built from `landing/Dockerfile` (private repo). Separate deployment on Dokploy.

---

## Database

PostgreSQL 16. Two EF Core contexts:

| Context | Connection string key | Manages |
|---|---|---|
| `CampaignDbContext` | `DefaultConnection` | Campaigns, members, sessions, maps, snapshots, roll history |
| `GamesDbContext` | `gamesdb` | Characters, inventory, wallet, progression |

Migrations run at application startup via `Use*Module()` in each module's extension - do not run `dotnet ef database update` manually.

The back CI pipeline (`back/.github/workflows/ci.yml`) runs a migration drift check on every push: `dotnet ef migrations has-pending-model-changes` for both `CampaignDbContext` and `GamesDbContext`. A drift would crash the container at startup; catching it in CI means it never reaches Dokploy.

Local dev database setup: see `back/README.md` (PostgreSQL 16 via `docker run`).

---

## Environment variables

### Backend

Set in Dokploy's UI vault for production. For local dev, set as persistent Windows user-level environment variables (not in `appsettings*.json` or `dotnet user-secrets`) - see `back/README.md` for the PowerShell one-liners. The double underscore in variable names maps to .NET's `:` configuration hierarchy separator (e.g. `Discord__ClientId` = `Discord:ClientId`).

| Variable | Required | Notes |
|---|---|---|
| `JWT_SECRET_KEY` | yes | Maps to `Jwt__SecretKey` |
| `JWT_ISSUER` | no | Default: `dndiscord-backend` |
| `JWT_AUDIENCE` | no | Default: `dndiscord-frontend` |
| `JWT_EXPIRATION_MINUTES` | no | Default: `10080` (7 days) |
| `DISCORD_CLIENT_ID` | yes | Maps to `Discord__ClientId` |
| `DISCORD_CLIENT_SECRET` | yes | Maps to `Discord__ClientSecret` |
| `DISCORD_REDIRECT_URI` | no | Default: `http://localhost:3000/auth/callback` |
| `CORS_ORIGIN` | no | Default: `http://localhost:3000` |

Connection strings (`DefaultConnection`, `gamesdb`) are configured separately in Dokploy; see `back/docker-compose.yml` for the local compose equivalents.

### Frontend

| Variable | Set at | Notes |
|---|---|---|
| `VITE_API_URL` | build time (Docker build arg) | Base URL of the backend API. Defaults to `http://localhost:5054` in local dev. Set to the production backend URL via Dokploy build args. |

---

## Nginx

The front container runs `nginx:alpine`. The config lives at `front/dndiscord-esp/nginx.conf`. It serves the Vite SPA with three cache tiers (immutable hashed bundles, 1-day must-revalidate for stable-filename assets, SPA fallback) and sets `Content-Security-Policy` to allow iframe embedding from `*.discordsays.com` and `*.discord.com`.

Before merging any change to `nginx.conf`, validate the config locally:

```sh
docker run --rm -v "$PWD/nginx.conf:/etc/nginx/conf.d/default.conf:ro" nginx:alpine nginx -t
```

The front CI pipeline runs this same check on every push. **Always quote regexes containing `{}`** in location blocks - an unquoted `{8,}` is parsed as a directive block by nginx and crash-loops the container (this has happened in production).

---

## Discord Activity setup

DnDiscord runs as a Discord Activity inside a voice channel iframe. The iframe sandbox forbids `eval`, `Function()`, inline blob workers, and all external CDNs - every font, icon, and script must be bundled or self-hosted, and BabylonJS must use `LoadAssetContainerAsync` rather than eval-based compilation paths.

URL mappings for the Activity are configured in the Discord Developer Portal and point at `dndiscord.cadran.app`. The nginx `Content-Security-Policy` header (`frame-ancestors 'self' https://*.discordsays.com ...`) is required for the iframe to load; `X-Frame-Options` is intentionally absent because sending it would cause the browser to reject the Discord iframe.
