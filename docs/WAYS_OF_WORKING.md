# DnDiscord - Ways of Working

How the team plans, syncs, codes, reviews and ships. This is the operational companion to [PROJECT_HISTORY.md](./PROJECT_HISTORY.md), which tells the story of how we got here.

> POC project, Epitech MSc Pro 2026 (T-ESP-902-96859). Lean by design. Read what you need, skip what you don't. Day-to-day implementation rules live in [CONTRIBUTING.md](./CONTRIBUTING.md), onboarding in [ONBOARDING.md](./ONBOARDING.md), delivery procedure in [DELIVERY_WORKFLOW.md](./DELIVERY_WORKFLOW.md).

---

## 1. Agile model

We use a lightweight, four-tier hierarchy adapted from the original Notion workspace and trimmed to what we actually use:

```
Theme  ->  Epic  ->  Feature  ->  Task
```

| Tier | What it is | Where it lives | Example |
|---|---|---|---|
| **Theme** | High-level domain. Stable across the project. | Implicit, named in PR scopes. | `multiplayer`, `campaign`, `progression`, `auth`, `dm-tools`, `engine`, `ci`, `delivery`. |
| **Epic** | Coherent body of work spanning multiple sprints. | Notion (legacy) `Découpage Epics Features`, GitHub PR titles, this document. | "Server-authoritative combat", "DM toolkit", "Campaign multi", "Discord Activity integration". |
| **Feature** | A user-visible capability you can demo. | One or more PRs targeted at `dev`. | "DM grants gold", "DM switches map", "Player declines roll request". |
| **Task** | Concrete unit of dev work. | Individual commits, optionally checklist items in a PR description. | "Add `DmGrantGold` hub method", "Wire `MapSwitched` event in front store". |

### Definition of Ready (DoR) - feature can start

A feature is ready to be picked up when:

- Theme and epic are clear, named in the branch / PR title.
- Acceptance criteria are explicit: what the user sees, what the API returns, what state changes.
- Boundary owners are identified (which module, which DbContext, which hub).
- Any cross-module bridge it needs is named (or the adapter pattern is followed - see [ARCHITECTURE.md](./ARCHITECTURE.md)).
- It is small enough to fit one PR. If not, it gets split before work starts.

### Definition of Done (DoD) - feature can ship

The full Notion DoD is documented under "Definition of Done (DoD)" in the legacy workspace and aspires to product-stage quality (80% test coverage, security audit, performance benchmarks at 1000+ concurrent users). For the POC we apply the **practical subset** that the CI and PR review actually enforce:

- Branch targets `dev`. PR title mirrors the primary commit subject.
- All CI checks green. No exceptions, no `--no-verify`.
- No regression in the existing test suites (back: xUnit + Testcontainers; front: vitest).
- New cross-module dependency uses the adapter pattern (no circular project references).
- New EF model change ships with its migration. CI runs `dotnet ef migrations has-pending-model-changes` for both `CampaignDbContext` and `GamesDbContext`.
- Frontend touch: `npx tsc -b` clean, `npx vitest run` green, `npx vite build` succeeds, `nginx -t` passes inside `nginx:alpine` if `nginx.conf` was touched.
- One human approval from `@aCuriousDev`.
- Demo-able end-to-end on the live `dev` deployment after merge.
- No secret committed (`Discord__*`, `JWT_SECRET_KEY`, real connection strings, `.env`, `appsettings.Development.json`).

The full Notion DoD remains the ambition for a productised v1.0; the practical subset is what gates merges today.

---

## 2. Sync cadence

### Weekly synchronous call

One scheduled voice call per week on Discord. Roughly 30-60 min. Standing agenda:

1. What shipped on `dev` since last week.
2. What is in flight / blocked.
3. Open architecture or scope decisions.
4. The next milestone (next school deliverable, next demo, next release).

This is where we make scope and architecture calls collectively. Async chat fills the gap the rest of the week.

### Daily-ish asynchronous chat

Discord is the team's command line. Channels in active use:

- **General project channel** - day-to-day questions, "I'm stuck on X", screenshots, decisions in writing.
- **`#code-reviews` style threads** - review pings, "PR ready", "fixed your comment".
- **Voice channels** - ad-hoc pair-debugging when chat is too slow.

Cadence is daily-ish, not enforced. The expectation is: if you are blocked or about to make a decision that affects someone else, say so in chat now, not on the next weekly sync.

### Pull request pings

When a PR is ready for review, the author pings `@aCuriousDev` (Quentin) directly in the relevant Discord channel with a link. This is the trigger for a review pass.

### Why we lean on PR review

PRs are the **primary quality gate** on this project, deliberately. Reviews tend to be thorough: the reviewer reads the diff, runs the live `dev` deploy after merge, and pushes back on:

- Hidden assumptions about state ("works in lobby, breaks in combat").
- Missing migration / missing test / missing nginx validation.
- Cross-module shortcuts that bypass the adapter pattern.
- Anything that would make the next contributor lose a day (Windows reserved filenames, unquoted nginx regex, `setStore({})` "clears", etc.).

A round of comments on a PR is normal and not a sign that something is wrong. The PR is where we maintain functionality and quality consistency, not the daily standup we do not have.

---

## 3. Branch model

| Branch | Role |
|---|---|
| `main` | Released / delivered. Tagged `v0.1.x`. Source for school mirroring. Per [DEPLOYMENT.md](./DEPLOYMENT.md), this is the release branch; current Dokploy convention auto-deploys `dev`. |
| `dev` | Integration branch. Auto-deployed to Dokploy on every push. Target for all PRs. Must stay in a working state. |
| `feature/<short-kebab>` | New functionality, branched from `dev`. |
| `fix/<short-kebab>` | Bug fix, branched from `dev`. |
| `hotfix/<short-kebab>` | Urgent fix that goes to `main` directly and is back-merged to `dev`. |
| `ci/<short-kebab>` | CI / workflow changes only. |
| `refactor/<short-kebab>` | Internal restructuring, no behaviour change. |
| `test/<short-kebab>` | Test additions or fixes. |
| `docs/<short-kebab>` | Documentation only. |

Real examples from the project:

```
feature/dm-roll-request
fix/level-up-ownership-check
hotfix/nginx-regex-syntax
ci/smoke-and-cache
refactor/split-dm-grant-broadcast
test/progression-asi-idempotency
docs/expand-readme-ci-section
```

Lowercase kebab-case. Keep it short enough to read in a `git branch` list.

---

## 4. Commit conventions

[Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <short imperative summary>
```

Common types: `feat`, `fix`, `docs`, `chore`, `ci`, `refactor`, `test`, `hotfix`, `i18n`, `merge`, `release`. Subject under 72 characters. Body is optional and reserved for the *why* when the summary alone does not make it obvious.

Real examples that ship in the history:

```
fix(security): require ownership for character level-up REST endpoint
feat(progression): ASI idempotency + AsiAppliedCount migration
ci: EF migration drift check + production image build
hotfix(nginx): quote regex with {8,} quantifier — front container crash-looped
refactor(dto): enum CurrencyType + drop GoldDelta + records
test(progression): ASI idempotency + tx rollback coverage
i18n(legal): translate /privacy /terms /legal /cookies /login to English
```

Treat commits as the team's voice. Author is the account pushing the branch.

---

## 5. Pull request flow

```
feature branch
    -> open PR against dev
       -> CI runs (back or front pipeline, see §6)
       -> ping @aCuriousDev on Discord
          -> review pass: read, deploy, comment
             -> author addresses comments
                -> approval + green CI
                   -> squash or merge to dev
                      -> Dokploy auto-deploys dev
                         -> manual smoke check on the live deploy
```

Rules:

- **Target:** `dev`, never `main` directly (except `hotfix/*` -> `main`, then back-merge).
- **Scope:** one concern per PR. A fix and a refactor go in two PRs.
- **Title:** mirrors the primary commit subject. Example: `fix(security): require ownership for character level-up REST endpoint`.
- **Body:** what changed, why it changed, brief test-plan checklist. Hub method changes name the hub, the methods, and the events broadcast.
- **CI:** all green before merge. A red PR does not get merged, period.
- **Review:** one approval from `@aCuriousDev` (project lead). Other reviewers welcome but not gating.
- **Merge:** standard merge or squash, author's choice depending on commit hygiene of the branch.

---

## 6. CI gates

### Backend (`epi-esp-back`) - job `build-and-test`

Triggers: push and PR to `main` and `dev`.

1. `dotnet restore`.
2. `dotnet build --no-restore`.
3. `dotnet test --no-build` - all unit and integration suites; integration tests use Testcontainers and require Docker on the runner (`ubuntu-latest` provides it).
4. **EF migration drift - `CampaignDbContext`** - `dotnet ef migrations has-pending-model-changes`. Catches "entity changed but migration forgotten" before Dokploy hits a startup crash.
5. **EF migration drift - `GamesDbContext`** - same check for the second context.
6. **Production image build** - `docker build -f Services.Dockerfile`. Validates the multi-stage build, published binary paths, runtime image. Does not start the container (no live DB at CI time).

### Frontend (`epi-esp-front`) - job `build`

Triggers: push and PR to `main` and `dev`.

1. **Cache `~/.npm`** - keyed on `package.json` content hash.
2. **Install** - `rm -f package-lock.json && npm install`. The committed lockfile is Windows-generated and is missing the Linux rollup binary (`@rollup/rollup-linux-x64-gnu`); CI installs fresh.
3. **Typecheck** - `npx tsc -b`. Catches type errors that Vite would silently pass through at build time.
4. **Test** - `npx vitest run`. Pure-logic suites: game rules, mappers, SignalR normalizers, pathfinding, etc.
5. **Build** - `npx vite build`. Produces the production bundle in `dist/`.
6. **Validate `nginx.conf`** - mounts the file into `nginx:alpine` and runs `nginx -t`. Catches unquoted regex quantifiers like `{8,}`.
7. **Production image smoke test** - builds the prod `Dockerfile`, starts the container on port 8080, polls `http://localhost:8080/` up to five times. Fails if the container never responds.

### Local checks before pushing

Run these before opening a PR to skip a CI round-trip:

```sh
# Backend
dotnet build
dotnet test                   # requires Docker - Testcontainers spins up PostgreSQL 16
# Unit-only, no Docker:
dotnet test --filter "FullyQualifiedName~Unit|FullyQualifiedName~Utils"
```

```sh
# Frontend
cd dndiscord-esp
npm run typecheck             # tsc -b
npm test                      # vitest run
# If you touched nginx.conf:
docker run --rm \
  -v $PWD/nginx.conf:/etc/nginx/conf.d/default.conf:ro \
  nginx:alpine nginx -t
```

Discord OAuth secrets for backend integration tests are persistent Windows user-level env vars (`Discord__ClientId`, `Discord__ClientSecret`, `Discord__RedirectUri`). Open a new terminal after setting them so the values are picked up. See [ONBOARDING.md](./ONBOARDING.md).

---

## 7. Delivery rhythm

DnDiscord ships in two layers:

### Layer 1 - children

Day-to-day delivery happens in `epi-esp-back` and `epi-esp-front`:

1. `dev` accumulates merged PRs and is continuously demo-able on Dokploy.
2. When `dev` is stable and ready for a release: `git checkout main && git merge --ff-only dev`.
3. `git tag v0.1.x` and `git push origin main && git push origin v0.1.x`.
4. The child's `bump-umbrella` workflow auto-bumps the umbrella's submodule pointer for that child.

### Layer 2 - umbrella + school mirror

The umbrella exists for school deliveries:

1. Auto-bump on each child push to `main` updates the umbrella's submodule pointer (3-attempt retry with rebase for race conditions). See [SUBMODULE_AUTOBUMP.md](./SUBMODULE_AUTOBUMP.md).
2. New deliverables (PDFs, slides, captures) get dropped into [`delivery/`](../delivery/) and committed.
3. Tag the umbrella delivery: `git tag -a delivery-<name> -m "Delivery: <milestone-name>"`.
4. `git push origin main --tags`.
5. The umbrella's `mirror-to-school` workflow auto-pushes `main` to `EpitechMscProPromo2026/T-ESP-902-96859-LYO_DnDiscord` via SSH (PATs are blocked by the school org's SAML enforcement). See [SCHOOL_MIRROR_AUTOMATION.md](./SCHOOL_MIRROR_AUTOMATION.md).

Manual fallbacks (when a workflow is broken or a PAT has expired) are documented in [DELIVERY_WORKFLOW.md](./DELIVERY_WORKFLOW.md).

End to end: child `main` push -> umbrella `main` updated -> school repo mirrored. The grader sees the same SHA the team tagged.

---

## 8. Tooling

| Tool | Used for | Notes |
|---|---|---|
| **Discord** | Sync (weekly call), async (daily-ish), PR pings, voice debug, the product itself. | Both the medium and the platform. |
| **GitHub** (`aCuriousDev` org) | Source of truth, PRs, CI, releases, auto-bump + mirror workflows. | Three child repos + umbrella. |
| **Dokploy** | Auto-deployment of `dev` for both `back` and `front`. | Watches `dev` branch via the GitHub App webhook. Production env vars in the Dokploy vault. |
| **Notion** (legacy workspace `T-ESP-800 - DnDiscord`) | Original PRD, SFD, SFR, DoD, Quality Plan, Cybersecurity Plan, Business Plan, Communication Strategy, follow-up notes. | Not actively maintained anymore; the source of truth is now this repo. See [PROJECT_HISTORY.md](./PROJECT_HISTORY.md) §2 for the plan-vs-reality reconciliation. |
| **Seq** | Backend log aggregation in Docker (`seq-net` network). Frontend bridges browser console via `/api/dev/log` in dev. | See [ARCHITECTURE.md](./ARCHITECTURE.md). |
| **Testcontainers (PostgreSQL 16)** | Backend integration tests. | Requires Docker running locally. |
| **Vitest** (pinned 3.x) | Frontend unit / pure-logic tests. | 4.x rolldown native binding broke on Windows at adoption time. |
| **Playwright / Chrome DevTools MCP** | Manual browser-side debugging during PR review. | Optional, used by the lead when reviewing front PRs that touch render or layout. |

---

## 9. Things we deliberately do not do

To keep the POC's scope honest:

- **No half-finished implementations on `dev`.** A feature lands behind a single PR or it stays on the feature branch.
- **No backwards-compatibility shims for unreleased internal APIs.** Rename and move on.
- **No microservices, no message bus, no caching layer.** Modular monolith. See [PROJECT_HISTORY.md](./PROJECT_HISTORY.md) §2 for the *why*.
- **No PR target other than `dev`** (except hotfix -> `main`).
- **No bypass of CI.** Failing CI = fix the cause, not the check.
- **No commit of secrets, ever.** `Discord__*`, `JWT_SECRET_KEY`, `appsettings.Development.json`, `.env`, `CLAUDE.md`, `.cursorrules`, `.claude/` - all gitignored, all stay gitignored.
- **No changes pushed to the school mirror directly.** It is derived. Always push to personal `origin` first.

For the implementation rules these conventions translate into in code, see [CONTRIBUTING.md](./CONTRIBUTING.md). For onboarding a new contributor, [ONBOARDING.md](./ONBOARDING.md). For the delivery procedure, [DELIVERY_WORKFLOW.md](./DELIVERY_WORKFLOW.md).
