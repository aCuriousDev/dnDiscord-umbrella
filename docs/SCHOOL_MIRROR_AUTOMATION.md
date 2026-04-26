# School Mirror Automation

A GitHub Actions workflow at [`.github/workflows/mirror-to-school.yml`](../.github/workflows/mirror-to-school.yml) mirrors every push to `main` of `aCuriousDev/dnDiscord-umbrella` over to `EpitechMscProPromo2026/T-ESP-902-96859-LYO_DnDiscord`.

## How it works

On `push` to `main` (or manual `workflow_dispatch`):

1. The runner does a bare clone of the umbrella over HTTPS using the auto-provided `GITHUB_TOKEN`.
2. It adds the school repo as a second remote, authenticated via the `SCHOOL_PAT` secret.
3. It runs `git push --mirror school`, which copies every ref (branches + tags) byte-for-byte.

Submodules are NOT cloned by the workflow. `git push --mirror` only ships the umbrella's own refs (gitlinks pointing at SHAs in the child repos). The school clone reaches the child repos directly when someone runs `git clone --recurse-submodules` against the school remote.

## Why a PAT and not a deploy key

The school repo is owned by the `EpitechMscProPromo2026` org. Adding a deploy key requires `admin` permission on that repo, which the project owner does not have (only `push`, `pull`, `triage`). Fine-grained PAT is the next least-privileged option.

## One-time setup

### 1. Mint a fine-grained PAT

1. Go to <https://github.com/settings/personal-access-tokens/new>.
2. Choose:
   - **Token name:** `dnDiscord-umbrella school mirror`
   - **Expiration:** 90 days (or however long the project still runs).
   - **Resource owner:** `aCuriousDev` (your personal account).
   - **Repository access:** "Only select repositories" -> pick `EpitechMscProPromo2026/T-ESP-902-96859-LYO_DnDiscord` (yes, you can target a repo you do not own as long as you have `push` on it).
   - **Repository permissions:**
     - `Contents`: **Read and write**
     - `Metadata`: Read (auto-selected)
3. Generate. Copy the token (`github_pat_...`) immediately, it is shown once.

### 2. Authorize the PAT for SAML SSO

The `EpitechMscProPromo2026` org enforces SAML. After generating the PAT:

1. Visit <https://github.com/settings/tokens?type=beta>.
2. Find the new token, click **Configure SSO**.
3. Authorize for `EpitechMscProPromo2026`.

Without this, the workflow gets `401` even with valid credentials.

### 3. Add the PAT as a repo secret

```bash
gh secret set SCHOOL_PAT -R aCuriousDev/dnDiscord-umbrella
# paste the token at the prompt
```

Or in the UI: <https://github.com/aCuriousDev/dnDiscord-umbrella/settings/secrets/actions>.

### 4. Verify

```bash
gh workflow run "Mirror to school" -R aCuriousDev/dnDiscord-umbrella
gh run watch -R aCuriousDev/dnDiscord-umbrella
```

Or just push any change to `main` and watch the run on the Actions tab.

## Disable temporarily

Set a repository variable to skip the job without deleting the workflow:

```bash
gh variable set MIRROR_ENABLED -R aCuriousDev/dnDiscord-umbrella --body "false"
```

Re-enable:

```bash
gh variable delete MIRROR_ENABLED -R aCuriousDev/dnDiscord-umbrella
```

## Rotate the PAT

PATs expire. When near expiry:

1. Mint a fresh PAT (same scopes as setup step 1).
2. Re-authorize SSO (step 2).
3. Overwrite the secret: `gh secret set SCHOOL_PAT -R aCuriousDev/dnDiscord-umbrella`.
4. Old PAT can be revoked at <https://github.com/settings/personal-access-tokens>.

## Troubleshooting

| Symptom in workflow log | Likely cause | Fix |
|---|---|---|
| `SCHOOL_PAT secret is not set` | Secret missing | Setup step 3. |
| `remote: Repository not found` over HTTPS | PAT not authorized for SAML | Setup step 2. |
| `remote: Permission to ... denied` | PAT scoped to wrong repo or missing `Contents: write` | Re-mint with correct scopes. |
| `error: failed to push some refs` with non-fast-forward | Someone pushed to school directly | Manual reconcile. School should never be pushed to outside this workflow. |
| Job is skipped (no run) | `MIRROR_ENABLED=false` repo variable set | `gh variable delete MIRROR_ENABLED ...` |

## Manual fallback

If the workflow is broken and a delivery is due, fall back to the manual procedure documented in [`DELIVERY_WORKFLOW.md`](./DELIVERY_WORKFLOW.md):

```bash
cd Y:/Dev Bis/dnDiscord-umbrella
git push --mirror school
```

This requires the local `school` remote to be set and your personal SSH key SSO-authorized for the `EpitechMscProPromo2026` org.

## Security notes

- The PAT lives only as a GitHub Actions secret. It is masked in logs and never written to disk by the workflow.
- The PAT is scoped to a single repo (`Contents: Read and write`). Compromise impact: someone could rewrite history on the school mirror, nothing else.
- The workflow uses `permissions: contents: read` at the job level, so the auto-provided `GITHUB_TOKEN` cannot modify the umbrella repo.
- The bare clone is in `$RUNNER_TEMP`-equivalent and discarded with the runner.
