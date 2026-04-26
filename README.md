# DnDiscord - Umbrella Repository

> Turn your Discord server into a D&D table. Stunning 3D adventures, no downloads, no extra accounts - just the magic of tabletop, shared with friends, anywhere.

Parent repo for the DnDiscord POC (Epitech MSc Pro 2026 - T-ESP-902-96859).
Wraps the backend, frontend, and landing-page repos as submodules, plus delivery and documentation folders.

**Current release:** `v0.1.1` (back + front).

## Layout

```
dnDiscord-umbrella/
├── back/        ← submodule → aCuriousDev/epi-esp-back     (C# / .NET, SignalR, EF Core)
├── front/       ← submodule → aCuriousDev/epi-esp-front    (SolidJS / TS / Tailwind)
├── landing/     ← submodule → aCuriousDev/epi-esp-landing  (PRIVATE - see below)
├── delivery/    ← final deliverables (pitch, comms, legal, playtest, demo)
├── docs/        ← project documentation (workflow, architecture)
└── README.md    ← this file
```

## Live Links

- **Live app** - <https://dndiscord.cadran.app/>
- **Landing page** - <https://dndiscord-landing.cadran.app/>
- **Instagram** - <https://www.instagram.com/p/DXmXkZbCuql/>
- **Reddit** - <https://www.reddit.com/r/Dndiscord_Off/>

## Cloning

```bash
git clone --recurse-submodules git@github.com:aCuriousDev/dnDiscord-umbrella.git
cd dnDiscord-umbrella
```

If you forgot `--recurse-submodules`:
```bash
git submodule update --init --recursive
```

> The `landing/` submodule points to a **private** repository. Without access, the
> landing checkout will fail - that is expected. Skip it with:
> ```bash
> git -c submodule.landing.update=none submodule update --init --recursive
> ```
> Live deployment of the landing page is available at
> <https://dndiscord-landing.cadran.app/>.

## Updating to latest delivered code

Submodules are pinned to specific commits. To pull the most recent main on each child:

```bash
git submodule update --remote --merge
```

The full delivery procedure (pointer bumps, tagging, mirror push) is documented in
[`docs/DELIVERY_WORKFLOW.md`](./docs/DELIVERY_WORKFLOW.md).

## Delivery materials

See [`delivery/`](./delivery/) for milestone deliverables:

- 50-word pitch (EN + FR)
- Communication strategy + visuals + channel captures
- Legal feasibility study (EN + FR)
- Playtest analytics

## Mirror

This repo is mirrored to the school repository at
`EpitechMscProPromo2026/T-ESP-902-96859-LYO_DnDiscord` at delivery milestones.
The personal `aCuriousDev/dnDiscord-umbrella` is the source of truth.

## Children repos

- Backend: <https://github.com/aCuriousDev/epi-esp-back>
- Frontend: <https://github.com/aCuriousDev/epi-esp-front>
- Landing (private): <https://github.com/aCuriousDev/epi-esp-landing> - live at <https://dndiscord-landing.cadran.app/>
