# Submodule Auto-Bump

A workflow on each child repo (`epi-esp-back`, `epi-esp-front`, `epi-esp-landing`) bumps the umbrella's submodule pointer whenever the child pushes to `main`. The umbrella push then chains into [`SCHOOL_MIRROR_AUTOMATION.md`](./SCHOOL_MIRROR_AUTOMATION.md), so the school repo updates without any manual step.

End-to-end flow:

```
child push to main
    -> child workflow `Bump umbrella submodule`
        -> SSH push of new gitlink to umbrella main
            -> umbrella workflow `Mirror to school`
                -> school repo synced
```

## Workflow location

`.github/workflows/bump-umbrella.yml` on each child repo. Identical content. Repository is identified at runtime via `${{ github.repository }}` and mapped to a submodule path:

| Child repo                     | Submodule path |
|--------------------------------|----------------|
| `aCuriousDev/epi-esp-back`     | `back`         |
| `aCuriousDev/epi-esp-front`    | `front`        |
| `aCuriousDev/epi-esp-landing`  | `landing`      |

## How the bump works

The workflow does not run `git submodule update --remote --merge`. It writes the gitlink directly:

```bash
git update-index --cacheinfo "160000,${NEW_SHA},${SUBMOD}"
```

This avoids cloning the submodule contents and avoids needing read access to the child from the deploy key. The umbrella does not need to have the child's commit in its object store: a gitlink is just a SHA reference, resolved by clients at clone time when they fetch the submodule from its own remote.

A 3-attempt retry with rebase-on-conflict handles the case where two children push to `main` simultaneously and race for the umbrella push.

## Why an SSH deploy key, not a PAT

Same reasoning as the school mirror: org-wide PAT policies are restrictive, and the children are owned by `aCuriousDev`, where deploy keys with write access are simple and least-privileged.

| Thing                          | Where                                            |
|--------------------------------|--------------------------------------------------|
| Deploy key (write)             | `aCuriousDev/dnDiscord-umbrella` repository keys |
| Same private half as secret    | `UMBRELLA_PUSH_KEY` on each of the 3 children    |
| Key label                      | `submodule-bump`                                 |

The same private key is reused across the 3 children to avoid managing 3 separate keys. All 4 repos belong to the same owner, so the threat model is uniform: any compromise of one child equals compromise of any other.

## One-time setup (already done for the current install)

```bash
# 1. Generate keypair locally.
ssh-keygen -t ed25519 -f /tmp/submodule_bump_key -N "" -C "submodule-bump"

# 2. Add public key as deploy key with write on umbrella.
gh repo deploy-key add /tmp/submodule_bump_key.pub --allow-write \
    -R aCuriousDev/dnDiscord-umbrella -t submodule-bump

# 3. Set the same private key as a secret on each child.
for r in aCuriousDev/epi-esp-back aCuriousDev/epi-esp-front aCuriousDev/epi-esp-landing; do
    gh secret set UMBRELLA_PUSH_KEY -R $r < /tmp/submodule_bump_key
done

# 4. Wipe the local copy.
rm -f /tmp/submodule_bump_key /tmp/submodule_bump_key.pub
```

The workflow file itself is delivered to each child via the standard PR flow (target `dev`, then promoted to `main` at release time).

## Activation

The workflow must exist on `main` of the child to fire on push to `main`. Sequence:

1. Merge `ci/bump-umbrella` PR to `dev` on the child (already open).
2. On next `dev -> main` release of the child, the workflow file lands on `main` and starts firing.
3. From then on, every release of that child auto-bumps the umbrella.

If you want activation immediately without waiting for a release, cherry-pick the workflow commit directly to `main` on the child.

## Disable temporarily

Per child, set a repo variable:

```bash
gh variable set UMBRELLA_BUMP_ENABLED -R aCuriousDev/epi-esp-back --body "false"
```

Re-enable:

```bash
gh variable delete UMBRELLA_BUMP_ENABLED -R aCuriousDev/epi-esp-back
```

## Rotate the key

```bash
# 1. Generate fresh keypair.
ssh-keygen -t ed25519 -f /tmp/submodule_bump_key -N "" -C "submodule-bump"

# 2. Replace the deploy key on the umbrella.
OLD_ID=$(gh repo deploy-key list -R aCuriousDev/dnDiscord-umbrella \
    --json id,title --jq '.[] | select(.title=="submodule-bump") | .id')
gh repo deploy-key add /tmp/submodule_bump_key.pub --allow-write \
    -R aCuriousDev/dnDiscord-umbrella -t submodule-bump
gh repo deploy-key delete "$OLD_ID" -R aCuriousDev/dnDiscord-umbrella

# 3. Re-set the secret on each child.
for r in aCuriousDev/epi-esp-back aCuriousDev/epi-esp-front aCuriousDev/epi-esp-landing; do
    gh secret set UMBRELLA_PUSH_KEY -R $r < /tmp/submodule_bump_key
done

# 4. Wipe the local copy.
rm -f /tmp/submodule_bump_key /tmp/submodule_bump_key.pub
```

## Troubleshooting

| Symptom in workflow log | Likely cause | Fix |
|---|---|---|
| `Permission denied (publickey)` on `git push` | `UMBRELLA_PUSH_KEY` empty or stale, deploy key removed | Re-set the secret. Confirm deploy key still on umbrella with write. |
| `failed to push some refs` after 3 attempts | High contention or umbrella protected branch | Manually run `workflow_dispatch` later. |
| Workflow runs but submodule not updated on umbrella | Concurrency lost; race resolved cleanly | No action; another run got there first with the same SHA. |
| Workflow does not fire | Workflow file not on `main` of child yet | Promote `dev -> main` on child or cherry-pick. |
| Job is skipped (no run) | `UMBRELLA_BUMP_ENABLED=false` | `gh variable delete UMBRELLA_BUMP_ENABLED -R <child>` |

## Manual fallback

If the auto-bump is broken on a child, do it manually from the umbrella:

```bash
cd Y:/Dev Bis/dnDiscord-umbrella
git submodule update --remote --merge -- <back|front|landing>
git add <back|front|landing>
git commit -m "chore(<sub>): bump"
git push origin main
```

The mirror-to-school workflow on the umbrella will then ship the change to the school repo automatically.
