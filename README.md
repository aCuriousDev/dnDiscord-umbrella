# DnDiscord — Umbrella Repository

Parent repo for the DnDiscord POC (Epitech MSc Pro 2026 — T-ESP-902-96859).
Wraps the backend, frontend, and landing-page repos as submodules, plus delivery and documentation folders.

## Layout

```
dnDiscord-umbrella/
├── back/        ← submodule → aCuriousDev/epi-esp-back     (C# / .NET)
├── front/       ← submodule → aCuriousDev/epi-esp-front    (SolidJS / TS)
├── landing/     ← submodule → aCuriousDev/epi-esp-landing  (PRIVATE — see below)
├── delivery/    ← final deliverables (reports, slides, demo)
├── docs/        ← project documentation
└── README.md    ← this file
```

## Live Links

- **Live app** — <https://dndiscord.cadran.app/>
- **Landing page** — <https://dndiscord-landing.cadran.app/>

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
> landing checkout will fail — that is expected. Skip it with:
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

## Mirror

This repo is mirrored to the school repository at
`EpitechMscProPromo2026/T-ESP-902-96859-LYO_DnDiscord` at delivery milestones.
The personal `aCuriousDev/dnDiscord-umbrella` is the source of truth.

## Children repos

- Backend: <https://github.com/aCuriousDev/epi-esp-back>
- Frontend: <https://github.com/aCuriousDev/epi-esp-front>
- Landing (private): <https://github.com/aCuriousDev/epi-esp-landing> — live at <https://dndiscord-landing.cadran.app/>
