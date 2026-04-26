# Onboarding - DnDiscord

Welcome. This guide gets a new contributor running locally in under an hour.

> POC project. Scope is intentionally lean. Read what you need, skip what you don't.

## 1. The big picture

DnDiscord is a [Discord Activity](https://discord.com/developers/docs/activities/overview) that turns a Discord voice channel into a 3D D&D table. Three repos make up the codebase, plus this umbrella.

| Repo | Stack | Purpose |
|---|---|---|
| `back/` | .NET 9, EF Core, SignalR, PostgreSQL | REST + real-time hub |
| `front/` | SolidJS, TypeScript, BabylonJS 7, Tailwind, nginx | Discord Activity iframe app |
| `landing/` | (private) | Marketing page at <https://dndiscord-landing.cadran.app/> |
| this umbrella | git submodules | Wraps everything for delivery |

Live app: <https://dndiscord.cadran.app/>

## 2. Access checklist

Before cloning, get:

- [ ] GitHub account added to `aCuriousDev` org collaborators (ask Quentin).
- [ ] SSH key registered on your GitHub account. Verify with `ssh -T git@github.com`.
- [ ] Access to the private `epi-esp-landing` repo if you'll touch the landing page.
- [ ] (Optional, only for delivery sync) Membership in `EpitechMscProPromo2026` GitHub org with SAML SSO authorized for your SSH key.

> HTTPS git through Git Credential Manager is blocked by Epitech's SAML enforcement. Use SSH for everything.

## 3. Tooling

Install once:

- **Git** 2.40+ (Windows: Git for Windows; macOS: brew; Linux: package manager)
- **.NET SDK 9** for `back/`
- **Node 20+** + **npm 10+** for `front/`
- **Docker Desktop** (Postgres + container builds)
- **PowerShell 7** (Windows) or any POSIX shell

Editor: VS Code or Rider. C# extension for `back`, Solid + ESLint extensions for `front`.

## 4. Clone

```bash
git clone --recurse-submodules git@github.com:aCuriousDev/dnDiscord-umbrella.git
cd dnDiscord-umbrella
```

Forgot the flag?

```bash
git submodule update --init --recursive
```

No access to `landing/`? Skip it:

```bash
git -c submodule.landing.update=none submodule update --init --recursive
```

## 5. First-time local run

Each child repo has its own dev guide. Follow them in order:

1. **Backend** - see `back/README.md` (sections "Local dev" and "Database"). You'll need PostgreSQL via `docker compose up -d`, then `dotnet ef database update` per context.
2. **Frontend** - see `front/README.md` (section "Local dev"). `npm ci && npm run dev`.

Discord OAuth secrets for local dev are kept as **persistent Windows user environment variables** (`Discord__ClientId`, etc.), not in `appsettings*.json` and not in `dotnet user-secrets`. Ask Quentin for values.

## 6. Branching and workflow

- `main` on every child repo = released / delivered code. Tagged (`v0.1.x`).
- `dev` = integration branch. Most PRs target `dev`.
- Feature branches: `feature/<short-name>`, `fix/<short-name>`, `hotfix/<short-name>`, `ci/<short-name>`.
- Open PRs against `dev`. CI runs build + tests + (front) nginx config check.
- Releases: fast-forward `dev` to `main`, tag `vX.Y.Z`, push tag.

Conventional Commits for messages: `feat(scope): ...`, `fix(scope): ...`, `docs(scope): ...`, `chore(scope): ...`, `ci(scope): ...`.

## 7. Delivery flow (umbrella)

Day-to-day work happens in the child repos. The umbrella exists for milestone delivery to the school. Full procedure: `docs/DELIVERY_WORKFLOW.md`.

Quick summary:

1. Promote `dev` to `main` on the children, tag.
2. In umbrella: `git submodule update --remote --merge` then commit pointer bumps.
3. Drop deliverables into `delivery/` (PDFs, slides, captures).
4. Push to personal `origin`, then mirror to school: `git push --mirror school`.

## 8. Gotchas

- **Discord Activity CSP** forbids `eval`, `Function()`, inline blob workers, external CDNs. Bundle or self-host everything. BabylonJS loaders use `LoadAssetContainerAsync` + instantiated templates.
- **SolidJS stores** with `createStore<Record<K,V>>`: `setStore({})` MERGES, it does not clear. Every clear helper must iterate keys and delete.
- **Windows reserved filenames**: `COM1.png` through `COM9.png` (also `LPT1-9`, `CON`, `PRN`, `AUX`, `NUL`) cannot be added by git on Windows. Use `Com01.png` style.
- **nginx config in `front/`**: regexes containing `{}` must be quoted. Always test locally with `docker run nginx:alpine nginx -t` before merging.
- **CLAUDE.md / .cursorrules / .claude/** are gitignored. Never commit them.

## 9. Where things live

| Need | Look here |
|---|---|
| Backend architecture | `back/README.md` |
| Frontend architecture | `front/README.md` |
| Delivery workflow | `docs/DELIVERY_WORKFLOW.md` |
| Final reports / pitch / comms | `delivery/` |
| Live app | <https://dndiscord.cadran.app/> |
| Landing page | <https://dndiscord-landing.cadran.app/> |
| Reddit | <https://www.reddit.com/r/Dndiscord_Off/> |
| Instagram | <https://www.instagram.com/p/DXmXkZbCuql/> |

## 10. Help

Stuck? Ping Quentin (project owner). For external context on D&D rules / Discord Activities, the docs in `back/README.md` and `front/README.md` cite primary sources.

Welcome aboard.
