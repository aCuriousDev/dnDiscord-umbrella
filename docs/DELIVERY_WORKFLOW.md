# Delivery Workflow

How to ship a milestone from the umbrella repo to the school repo.

> The school mirror is **automated** via GitHub Actions: every push to `main` on `aCuriousDev/dnDiscord-umbrella` triggers a mirror push to the school repo. See [`SCHOOL_MIRROR_AUTOMATION.md`](./SCHOOL_MIRROR_AUTOMATION.md). The manual `git push --mirror school` step below is the fallback path if the workflow is broken or the PAT has expired.

## Prerequisites

- `main` on `epi-esp-back` and `epi-esp-front` reflects the state to deliver.
- Both child repos pushed to `origin` (your personal `aCuriousDev/*` remotes).
- Any reports / PDFs / slides for this milestone copied into `delivery/`.

## Steps

```bash
cd "Y:/Dev Bis/dnDiscord-umbrella"

# 1. Pull umbrella latest (in case you updated it elsewhere)
git pull --recurse-submodules

# 2. Bump submodule pointers to latest main on both children
git submodule update --remote --merge

# 3. (Optional) Drop new delivery files into delivery/ at this point
#    Example: cp ../report-final.pdf delivery/

# 4. Stage and commit pointer bumps + delivery additions
git add back front delivery
git status   # sanity check
git commit -m "deliver: <milestone-name>"

# 5. Tag the delivery
git tag -a delivery-<name> -m "Delivery: <milestone-name>"

# 6. Push to personal origin (the GH Actions workflow takes it from here)
git push origin main --tags

# 7. (Manual fallback only) Mirror to school directly
#    Skip this if the GH Actions workflow is healthy. Use only when the
#    automation is broken or you need an out-of-band push.
# git push --mirror school
```

## Notes

- `git push --mirror` is destructive on the target. Always push to `origin` first; treat `school` as derived.
- Submodules pin to commit SHAs, not branches. `branch = main` in `.gitmodules` is metadata only - bump is explicit via `--remote`.
- Graders cloning the school repo MUST use `--recurse-submodules` (documented in root README).
- HTTPS git access to the school repo is blocked by SAML SSO. SSH works because the SSH key is enrolled separately. If HTTPS is needed, authorize Git Credential Manager for the `EpitechMscProPromo2026` org via GitHub web settings.
