# release-plz-app-token

Composite action that mints a short-lived GitHub App installation
token for `release-plz` and configures `git` + the `gh` CLI to act
under the App's bot identity.

## Why this exists

`release-plz` PRs opened with the workflow's default `GITHUB_TOKEN`
do **not** trigger downstream `pull_request`-keyed workflows. This is
a GitHub security feature (workflow-token pushes are intentionally
suppressed to prevent recursive workflow runs), but it breaks any
release flow that needs CI to run on a release-plz-opened PR — for
example, a downstream sync job that updates a co-released artifact's
metadata before merge.

The supported workaround is to mint an installation token from a
GitHub App and use that token for `release-plz` operations. App-token
pushes DO trigger `pull_request: synchronize`, so downstream jobs
fire as expected.

This action wraps the token mint + git/gh identity configuration so
consumer repos don't have to repeat the same three steps in every
workflow.

## Inputs

| Name | Required | Description |
|---|---|---|
| `client-id` | yes | Identifier for the org-wide release-plz App. Accepts either the App's **Client ID** (canonical form, e.g. `Iv23li...`) OR the App's **numeric App ID** — `@octokit/auth-app`'s `appId` parameter resolves both. Read from an org-level secret (e.g., `RELEASE_PLZ_APP_ID`); see *Secret name vs. input name* below for why the secret name need not change when its value is the integer form. |
| `private-key` | yes | App private key (PEM). Read from an org-level secret (e.g., `RELEASE_PLZ_APP_PRIVATE_KEY`). |
| `permission-contents` | yes | Token scope for the `contents` permission. Declare `read` or `write` explicitly; least-privilege is the safe default. |
| `permission-pull-requests` | no | Token scope for the `pull-requests` permission. Omit to inherit the App's installed permission, or set `read` / `write` to narrow. |

### Secret name vs. input name

The composite's `client-id` input is named after the canonical form `actions/create-github-app-token` v3.1.1+ prefers. The org-level secret holding the value, however, can legitimately be named anything — e.g. `RELEASE_PLZ_APP_ID` (with the integer form as its value) works exactly as `RELEASE_PLZ_APP_CLIENT_ID` (with the string form) would. The input cares about what the value resolves to, not what the secret is called. Renaming the secret is a no-op operationally and not required by this composite.

## Outputs

| Name | Description |
|---|---|
| `token` | The minted installation token. Pass to downstream steps via `env: GITHUB_TOKEN` / `env: GH_TOKEN`. |
| `app-slug` | The App's globally-assigned slug. See *Dynamic slug contract* below. |

## Usage

```yaml
jobs:
  release-pr:
    runs-on: ubuntu-latest
    environment: release-plz
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@<SHA> # vN
        with:
          persist-credentials: false

      - name: Mint release-plz App token
        id: app-token
        uses: breezy-bays-labs/.github/actions/release-plz-app-token@<TAG_SHA> # release-plz-app-token-vX.Y.Z
        with:
          client-id: ${{ secrets.RELEASE_PLZ_APP_ID }}
          private-key: ${{ secrets.RELEASE_PLZ_APP_PRIVATE_KEY }}
          permission-contents: write
          permission-pull-requests: write

      - name: Run release-plz
        uses: release-plz/action@<SHA> # vN
        env:
          GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
        with:
          command: release-pr
```

## Dynamic slug contract

GitHub App names must be globally unique. If `release-plz` was
already registered when the org App was created, GitHub appended a
disambiguator (e.g., `release-plz-1234`). The App's actual slug is
exposed at `steps.<id>.outputs.app-slug`.

**Consumers MUST reference the slug dynamically.** Hardcoding
`release-plz[bot]` in workflows or downstream scripts silently dies
when the actual slug differs.

```yaml
# correct
- run: echo "Author was ${{ steps.app-token.outputs.app-slug }}[bot]"

# wrong — silently fails if slug got disambiguated
- run: echo "Author was release-plz[bot]"
```

The composite uses the slug internally when setting the git
`user.name` / `user.email`, so commit attribution stays correct even
if the App is later renamed or re-created with a different slug.

## Pin discipline

Every consumer of this composite **must** SHA-pin the reference and
suffix a comment naming the human-readable tag, per the org's
supply-chain hygiene convention:

```yaml
uses: breezy-bays-labs/.github/actions/release-plz-app-token@<40-char SHA> # release-plz-app-token-v0.1.0
```

The trailing comment lets reviewers recognize the version without a
`gh api` lookup. SHA pinning guards against tag-poisoning attacks
(published tags are mutable; commit SHAs are not).

Configure Dependabot's `github-actions` ecosystem on the consumer
repo to track this pin — bumps land as PRs when a new
`release-plz-app-token-vX.Y.Z` tag is published from this repo.

## Allowlist

The canonical list of repos authorized to consume this composite (and
the org-level `RELEASE_PLZ_APP_*` secrets) lives at the repo root in
[`release-plz-allowlist.toml`](../../release-plz-allowlist.toml).

Consumer repos may run a lightweight CI lint that asserts their own
name appears in that allowlist — this catches drift in either
direction (a repo's name dropped from the canonical list, or a repo
trying to consume the secrets without being on the list).

## Security boundary

- The App private key never leaves the secret store. The minted
  token's lifetime is bounded by the App's standard installation-token
  TTL (one hour).
- `permission-contents` is required so callers can't accidentally
  mint a write-scoped token for a read-only job.
- The composite configures `git` globally on the runner. The runner
  is ephemeral (one job per VM), so the configuration is scoped to
  the run.
- This action does not write the private key to disk, log the minted
  token, or persist credentials via `actions/checkout` (consumers are
  expected to set `persist-credentials: false` on their checkout
  step).
