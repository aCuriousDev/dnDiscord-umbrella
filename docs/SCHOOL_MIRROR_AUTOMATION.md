# School Mirror Automation

A GitHub Actions workflow at [`.github/workflows/mirror-to-school.yml`](../.github/workflows/mirror-to-school.yml) mirrors every push to `main` of `aCuriousDev/dnDiscord-umbrella` over to `EpitechMscProPromo2026/T-ESP-902-96859-LYO_DnDiscord`.

## How it works

On `push` to `main` (or manual `workflow_dispatch`):

1. `webfactory/ssh-agent` loads the dedicated SSH key stored as the `SCHOOL_SSH_KEY` secret into the runner's ssh-agent.
2. The runner does a bare HTTPS clone of the umbrella using the auto-provided `GITHUB_TOKEN`.
3. It adds the school repo as an SSH remote and runs `git push --mirror school`. The push uses the SSH key registered on the project owner's GitHub account.

Submodules are NOT cloned by the workflow. `git push --mirror` only ships the umbrella's own refs (gitlinks pointing at SHAs in the child repos). The school clone reaches the child repos directly when someone runs `git clone --recurse-submodules` against the school remote.

## Why an SSH key, not a PAT or deploy key

| Option | Why ruled out |
|---|---|
| Fine-grained PAT | The `EpitechMscProPromo2026` org policy blocks PATs targeting org-owned repos. |
| Deploy key on the school repo | Adding a deploy key needs `admin` on that repo. The project owner only has `push`. |
| GitHub App | Requires org admin to install. Overkill for POC. |
| **Dedicated SSH key on owner's account** | Works because SSH bypasses SAML once the key is SSO-authorized. Least friction available. |

The trade-off: a personal SSH key is account-wide, not repo-scoped. It can push to any repo the owner can push to. Mitigations:

- It is a dedicated key (named `umbrella-mirror`), not the owner's primary key. Easy to identify and revoke without touching daily workflows.
- The private half lives only as a GitHub Actions secret, masked in logs, never written to disk by the workflow.
- The workflow uses `permissions: contents: read` at the job level, so the auto-provided `GITHUB_TOKEN` cannot modify the umbrella repo.

## One-time setup (already done for current install)

The `umbrella-mirror` SSH key was generated, the public half registered on the project owner's GitHub account, and the private half stored as the `SCHOOL_SSH_KEY` secret on `aCuriousDev/dnDiscord-umbrella`. The remaining step the owner had to do interactively was the SSO authorization in the browser - covered in the next section in case the key needs to be re-authorized.

## Activate or re-authorize SSO

After registering the key (or any time the org SSO authorization for it lapses):

1. Open <https://github.com/settings/keys>.
2. Find `umbrella-mirror` in the SSH keys list.
3. Click **Configure SSO** -> **Authorize** for `EpitechMscProPromo2026`.

Without this, the workflow gets `Permission denied (publickey)` even with a valid key.

## Verify

```bash
gh workflow run "Mirror to school" -R aCuriousDev/dnDiscord-umbrella
gh run watch -R aCuriousDev/dnDiscord-umbrella
```

Or push any change to `main` and watch the run on the Actions tab. A green run means the school repo is now in sync with `aeed81e` or whatever the current `main` HEAD is.

## Disable temporarily

Set a repository variable to skip the job without deleting the workflow:

```bash
gh variable set MIRROR_ENABLED -R aCuriousDev/dnDiscord-umbrella --body "false"
```

Re-enable:

```bash
gh variable delete MIRROR_ENABLED -R aCuriousDev/dnDiscord-umbrella
```

## Rotate the SSH key

The key has no expiry. Rotate when:

- The owner suspects the secret leaked.
- The key needs broader or narrower access.
- An audit calls for it.

```bash
# 1. Generate fresh key (no passphrase, dedicated label).
ssh-keygen -t ed25519 -f /tmp/umbrella_mirror_key -N "" -C "umbrella-mirror"

# 2. Register new public key.
gh ssh-key add /tmp/umbrella_mirror_key.pub --title "umbrella-mirror"

# 3. Re-authorize SSO (browser, see "Activate or re-authorize SSO" above).

# 4. Update the secret.
gh secret set SCHOOL_SSH_KEY -R aCuriousDev/dnDiscord-umbrella < /tmp/umbrella_mirror_key

# 5. Wipe the local copy.
rm -f /tmp/umbrella_mirror_key /tmp/umbrella_mirror_key.pub

# 6. Delete the OLD key from GitHub.
gh ssh-key list --json id,title | jq -r '.[] | select(.title=="umbrella-mirror-old") | .id' | xargs -I{} gh ssh-key delete {}
```

To find the old key's ID before rotation:

```bash
gh api user/keys --jq '.[] | select(.title=="umbrella-mirror") | {id, created_at}'
```

## Why the workflow guards on `github.repository`

`git push --mirror` copies every ref including `refs/heads/main`, which means the `.github/workflows/` directory is replicated to the school repo. Without a guard, the school repo's copy of `mirror-to-school.yml` would fire on every mirror push, fail because `SCHOOL_SSH_KEY` is not set there, and pile up red runs.

The job is gated by:

```yaml
if: ${{ github.repository == 'aCuriousDev/dnDiscord-umbrella' && vars.MIRROR_ENABLED != 'false' }}
```

When the workflow runs on the school clone (`github.repository == 'EpitechMscProPromo2026/...'`), the job is skipped cleanly and the run shows as "skipped" rather than "failed". Disabling Actions outright on the school repo would be cleaner but requires `admin` on that repo, which the project owner does not have.

## Troubleshooting

| Symptom in workflow log | Likely cause | Fix |
|---|---|---|
| `Error loading key: invalid format` (ssh-agent step) | `SCHOOL_SSH_KEY` secret was set with a malformed value (extra newline, BOM, public half pasted) | Re-set with `gh secret set SCHOOL_SSH_KEY -R aCuriousDev/dnDiscord-umbrella < <private-key-file>`. |
| `Permission denied (publickey)` on push | SSO not authorized for the key on the org | Authorize at <https://github.com/settings/keys>. |
| `Repository not found` | Org admin removed write access for `aCuriousDev` on the school repo | Coordinate with Epitech to restore. Until then, fall back to manual mirror. |
| `error: failed to push some refs` non-fast-forward | Someone pushed to school directly | Manual reconcile. School should never be pushed to outside this workflow. |
| Job is skipped (no run) | `MIRROR_ENABLED=false` repo variable is set | `gh variable delete MIRROR_ENABLED ...` |

## Manual fallback

If the workflow is broken and a delivery is due, fall back to the manual procedure documented in [`DELIVERY_WORKFLOW.md`](./DELIVERY_WORKFLOW.md):

```bash
cd Y:/Dev Bis/dnDiscord-umbrella
git push --mirror school
```

This requires the local `school` remote to be set and the owner's primary SSH key SSO-authorized for the `EpitechMscProPromo2026` org.
