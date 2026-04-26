# Documentation

Project documentation about the DnDiscord POC: architecture notes, setup guides, API references, design decisions.

For final delivery artifacts (reports, slides) see `delivery/`. For source code see `back/` and `front/`.

## Contents

- [`ONBOARDING.md`](./ONBOARDING.md) - new contributor guide. Start here.
- [`ARCHITECTURE.md`](./ARCHITECTURE.md) - umbrella-level system overview, data flow, component map.
- [`DEPLOYMENT.md`](./DEPLOYMENT.md) - Dokploy hosts, environments, env vars, container layout.
- [`CONTRIBUTING.md`](./CONTRIBUTING.md) - branch model, commit conventions, PR flow, CI checks.
- [`GLOSSARY.md`](./GLOSSARY.md) - project-specific terms (DM, Hub, Activity, ASI, etc.) with code pointers.
- [`DESIGN_SYSTEM.md`](./DESIGN_SYSTEM.md) - tokens, components, typography, motion, iconography, brand-asset prompts.
- [`DELIVERY_WORKFLOW.md`](./DELIVERY_WORKFLOW.md) - milestone delivery procedure to the school repo.
- [`SCHOOL_MIRROR_AUTOMATION.md`](./SCHOOL_MIRROR_AUTOMATION.md) - GitHub Actions workflow that auto-mirrors `main` to the school repo, plus key setup and rotation.
- [`SUBMODULE_AUTOBUMP.md`](./SUBMODULE_AUTOBUMP.md) - workflow on each child repo that auto-bumps the umbrella submodule pointer on push to `main`. Chains into the school mirror.
