# Contributing to DnDiscord

Quick reference for opening your first PR. POC project - keep things minimal.

---

## Where work happens

Day-to-day development happens in the two child repos:

- [`epi-esp-back`](https://github.com/aCuriousDev/epi-esp-back) - .NET 9 backend
- [`epi-esp-front`](https://github.com/aCuriousDev/epi-esp-front) - SolidJS frontend

The umbrella repo (`dnDiscord-umbrella`) is updated only at delivery milestones to advance submodule pointers. Do not open feature PRs here. See [docs/DELIVERY_WORKFLOW.md](DELIVERY_WORKFLOW.md) for the umbrella release flow.

---

## Branch model

| Branch | Role |
|---|---|
| `main` | Released / delivered. Tagged `v0.1.x`. Dokploy deploys from here in production. |
| `dev` | Integration branch. Auto-deployed to Dokploy on every push. Target for all PRs. |
| `feature/<short-kebab>` | New functionality, branched from `dev`. |
| `fix/<short-kebab>` | Bug fixes, branched from `dev`. |
| `hotfix/<short-kebab>` | Urgent fixes that go to `main` directly and are back-merged to `dev`. |
| `ci/<short-kebab>` | Pipeline / workflow changes only. |
| `refactor/<short-kebab>` | Internal restructuring, no behaviour change. |
| `test/<short-kebab>` | Test additions or fixes. |
| `docs/<short-kebab>` | Documentation only. |

Dokploy watches `dev` and redeploys automatically on every push. Keep `dev` in a working state at all times.

---

## Branch naming

Match the prefix to the type of change:

```
feature/dm-roll-request
fix/level-up-ownership-check
hotfix/nginx-regex-syntax
ci/smoke-and-cache
refactor/split-dm-grant-broadcast
test/progression-asi-idempotency
docs/expand-readme-ci-section
```

Use lowercase kebab-case. Keep it short enough to read in a branch list.

---

## Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/). Format:

```
<type>(<scope>): <short imperative summary>
```

Common types: `feat`, `fix`, `docs`, `chore`, `ci`, `refactor`, `test`, `hotfix`, `i18n`, `merge`, `release`.

Real examples from this project:

```
fix(security): require ownership for character level-up REST endpoint
feat(progression): ASI idempotency + AsiAppliedCount migration
ci: EF migration drift check + production image build
hotfix(nginx): quote regex with {8,} quantifier — front container crash-looped
refactor(dto): enum CurrencyType + drop GoldDelta + records
test(progression): ASI idempotency + tx rollback coverage
chore(gitignore): ignore .claude/ to prevent local Claude Code settings from leaking
i18n(legal): translate /privacy /terms /legal /cookies /login to English
```

Keep the subject line under 72 characters. Body is optional - use it when the why is not obvious from the summary alone.

Do not add `Co-Authored-By` lines or AI-generated attribution.

---

## Pull requests

- **Target branch:** always `dev`, never `main` directly.
- **Scope:** one concern per PR. A fix and a refactor belong in separate PRs.
- **Title:** mirrors the primary commit subject (`fix(security): require ownership for character level-up REST endpoint`).
- **Body:** include what changed, why it changed, and a brief test plan checklist.
- **CI:** all checks must be green before merge. Do not merge a red PR.
- **Review:** ping `@aCuriousDev` (Quentin Berger). One approval required.

---

## CI checks

### Backend (`epi-esp-back`) - job `build-and-test`

Triggered on push and pull request to `main` and `dev`.

- **Restore** - `dotnet restore`
- **Build** - `dotnet build --no-restore`
- **Test** - `dotnet test --no-build` (runs all unit and integration suites; integration tests use Testcontainers and require Docker on the runner, which `ubuntu-latest` provides)
- **EF migration drift - CampaignDbContext** - `dotnet ef migrations has-pending-model-changes` for `CampaignDbContext`. Catches "entity changed but migration forgotten" before Dokploy hits a startup crash.
- **EF migration drift - GamesDbContext** - same check for `GamesDbContext`.
- **Production image build** - `docker build -f Services.Dockerfile`. Validates the multi-stage build, published binary paths, and runtime image without starting the container (a live DB is required at runtime, which CI does not have).

### Frontend (`epi-esp-front`) - job `build`

Triggered on push and pull request to `main` and `dev`.

- **Cache `~/.npm`** - keyed on `package.json` content hash; avoids redundant network downloads across runs.
- **Install** - deletes `package-lock.json` then runs `npm install` fresh (the committed lockfile is Windows-generated and lacks the Linux rollup binary).
- **Typecheck** - `npx tsc -b`. Catches type errors that Vite would silently pass through at build time.
- **Test** - `npx vitest run`. Runs all pure-logic suites (game rules, mappers, SignalR normalizers, pathfinding).
- **Build** - `npx vite build`. Produces the production bundle in `dist/`.
- **Validate nginx.conf** - mounts `nginx.conf` into an `nginx:alpine` container and runs `nginx -t`. Catches syntax errors (notably unquoted regex quantifiers like `{8,}`) before they crash-loop the production container.
- **Production image smoke test** - builds the production `Dockerfile`, starts the container on port 8080, polls `http://localhost:8080/` up to five times. Fails if the container never responds.

---

## Local checks before pushing

Run these before opening a PR to avoid a CI round-trip.

### Backend

```sh
dotnet build
dotnet test        # requires Docker - Testcontainers spins up PostgreSQL 16
```

Integration tests need the Discord secrets to be set as persistent Windows user environment variables (`Discord__ClientId`, `Discord__ClientSecret`, `Discord__RedirectUri`). Open a new terminal after setting them so the values are picked up. See the backend README for the one-time PowerShell setup.

Unit tests only (no Docker):

```sh
dotnet test --filter "FullyQualifiedName~Unit|FullyQualifiedName~Utils"
```

### Frontend

```sh
cd dndiscord-esp
npm run typecheck   # tsc -b
npm test            # vitest run
```

If you touched `nginx.conf`, validate it locally before pushing:

```sh
docker run --rm \
  -v $PWD/nginx.conf:/etc/nginx/conf.d/default.conf:ro \
  nginx:alpine nginx -t
```

Quote any regex containing `{n,}` quantifiers. An unquoted brace is parsed as a directive block and will crash the container (this happened in production - see commit `01fb4f6d`).

---

## Releasing (children)

> Auto-bump: once the `bump-umbrella` workflow lives on a child's `main` branch, any push to that `main` automatically updates the umbrella's submodule pointer and (via the umbrella) the school mirror. See [`SUBMODULE_AUTOBUMP.md`](./SUBMODULE_AUTOBUMP.md). The steps below remain the manual fallback.



When `dev` is stable and ready to ship:

1. Fast-forward `main` to `dev`: `git checkout main && git merge --ff-only dev`
2. Tag the release: `git tag v0.1.x`
3. Push both: `git push origin main && git push origin v0.1.x`
4. Ping Quentin (`@aCuriousDev`) to bump the umbrella submodule pointer to the new tag and record the delivery in the umbrella changelog.

Dokploy auto-deploys on the push to `main`.

---

## Things that are gitignored - never commit

- `.claude/` and `CLAUDE.md` - local Claude Code settings
- `.cursorrules` - local Cursor IDE rules
- `.env` and any file containing secrets
- `appsettings.Development.json`
- `appsettings.*.json` with real connection strings or keys
- Discord OAuth credentials (`Discord__ClientId`, `Discord__ClientSecret`, `Discord__RedirectUri`) - these live as Windows user environment variables, not in files
- Any file containing `JWT_SECRET_KEY` or similar runtime secrets

If you accidentally stage a secret, remove it from history before pushing.
