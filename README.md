# Release on Merge Action

Github action to create git tag and Github release on merge.

## Usage

### Example

``` yaml
on:
  push:
    branches:
      - main

jobs:
  release-on-merge:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: dexwritescode/release-on-merge-action@v1
```

The only permissions needed for `GITHUB_TOKEN` to create tags and releases is `contents: write`.

## Inputs

### `version-increment-strategy`

**Optional** The version number to increment. Default `patch`.

It can be one of: `major`, `minor`, `patch`, `norelease`

```yml
version-increment-strategy: major
```

### `initial-version`

**Optional** If no previous release is found in the repo, the action will create the first release using this version. Default `0.1.0`.

```yml
initial-version: '1.2.3'
```

### `tag-prefix`

**Optional** Git tag prefix. Example the `v` in `v1.2.3`. Default `v`.

```yml
tag-prefix: v
```

### `body`

**Optional** Body text to prepend to the auto-generated release notes.

```yml
body: 'Body text...'
```

### `generate-release-notes`

**Optional** Whether to auto-generate Github release notes. Default `true`.

```yml
generate-release-notes: true
```

### `dry-run`

**Optional** Log the version that would be created without actually creating the release. Default `false`.

```yml
dry-run: true
```

### `github-host`

**Optional** GitHub API host. Override for GitHub Enterprise Server. Default `https://api.github.com`.

```yml
github-host: 'https://github.example.com/api/v3'
```

### `prerelease`

**Optional** Create a pre-release instead of a stable release. The version will include a pre-release identifier and an auto-incrementing counter (e.g. `v1.2.0-rc.1`, `v1.2.0-rc.2`). Default `false`.

```yml
prerelease: true
```

When `prerelease: true`:
- If the latest release already has the same identifier, the counter is incremented (`v1.2.0-rc.1` → `v1.2.0-rc.2`).
- Otherwise, `version-increment-strategy` is applied to the base version and the counter starts at `.1`.

### `prerelease-identifier`

**Optional** The identifier to append when `prerelease: true`. Default `rc`.

```yml
prerelease-identifier: beta
```

Produces tags like `v1.2.0-beta.1`, `v1.2.0-beta.2`, etc.

### `use-label-strategy`

**Optional** Derive the increment strategy from the merged PR's labels instead of `version-increment-strategy`. If no matching label is found, falls back to `version-increment-strategy`. Default `false`.

```yml
use-label-strategy: true
```

Default label names (all configurable — see below):

| Label | Strategy |
|---|---|
| `release:major` | major |
| `release:minor` | minor |
| `release:patch` | patch |
| `release:skip` | norelease (no release created) |

### `label-major`

**Optional** PR label that triggers a major version bump. Default `release:major`.

```yml
label-major: 'bump:major'
```

### `label-minor`

**Optional** PR label that triggers a minor version bump. Default `release:minor`.

```yml
label-minor: 'bump:minor'
```

### `label-patch`

**Optional** PR label that triggers a patch version bump. Default `release:patch`.

```yml
label-patch: 'bump:patch'
```

### `label-skip`

**Optional** PR label that skips release creation. Default `release:skip`.

```yml
label-skip: 'bump:skip'
```

### `version-files`

**Optional** Where the version lives in the repository, so the action keeps it in step with the tags. Default `''` (only tag, as without this input).

- `cargo`: the version in the root `Cargo.toml` (`[workspace.package]` or `[package]`), and the matching entries in `Cargo.lock`.
- `file:PATH`: a file holding only the version, e.g. `file:VERSION`.

```yml
version-files: cargo
```

See [Versions in files](#versions-in-files). Not supported with `prerelease`.

## Outputs

### `version`

The version number that was created (without prefix), e.g. `1.2.3` or `1.2.3-rc.1`.

### `tag`

The full tag that was created (with prefix), e.g. `v1.2.3` or `v1.2.3-rc.1`.

```yml
- uses: dexwritescode/release-on-merge-action@v1
  id: release
- run: echo "Created ${{ steps.release.outputs.tag }}"
```

## Advanced examples

### Label-driven strategy

```yaml
on:
  push:
    branches:
      - main

jobs:
  release-on-merge:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: dexwritescode/release-on-merge-action@v1
        with:
          use-label-strategy: true
```

Label your PRs with `release:major`, `release:minor`, `release:patch`, or `release:skip` to control the release type per merge.

### Pre-release workflow

```yaml
- uses: dexwritescode/release-on-merge-action@v1
  with:
    prerelease: true
    prerelease-identifier: beta
```

### Versions in files

Like Maven's `-SNAPSHOT`, the branch always holds the next version with `-dev`, and each release writes its exact version into the files:

1. A PR is merged while the files say `0.5.5-dev`. Its label picks the release, say `v0.5.5`.
2. The action adds a commit that sets the files to `0.5.5` and tags it `v0.5.5`, then a commit that sets them to `0.5.6-dev` (the release's patch + 1), and fast-forwards the branch.

So a tagged commit always holds its own version, and every other build reports a version above every release and below the one it will become. Any label works: `0.5.6-dev` followed by a `release:minor` PR releases `0.6.0` and moves to `0.6.1-dev`. The files may hold any version at or below the next release; above it, the action stops with an error.

Only pushes that merge a PR release, so the action's own commits never do. The token must be able to push to the branch; with a ruleset or branch protection that requires PRs, use a GitHub App that is on the bypass list:

1. Create a GitHub App (Settings → Developer settings → GitHub Apps) with no webhook and the repository permissions **Contents: Read and write** and **Pull requests: Read-only**. Install it on the repository.
2. Add the App to the branch's ruleset bypass list, mode **Always**.
3. Store its Client ID (on the App's settings page) as the variable `RELEASE_APP_CLIENT_ID` and a private key as the secret `RELEASE_APP_PRIVATE_KEY`.

```yaml
on:
  push:
    branches:
      - main

jobs:
  release-on-merge:
    # The action's own push starts this workflow again; skip it here so it never
    # joins the concurrency group, where it would cancel a waiting release.
    if: ${{ !startsWith(github.event.head_commit.message, 'Next development version') }}
    concurrency:
      group: release
      cancel-in-progress: false
    runs-on: ubuntu-latest
    steps:
      - uses: actions/create-github-app-token@v3
        id: app
        with:
          client-id: ${{ vars.RELEASE_APP_CLIENT_ID }}
          private-key: ${{ secrets.RELEASE_APP_PRIVATE_KEY }}
      - uses: dexwritescode/release-on-merge-action@v1
        env:
          GITHUB_TOKEN: ${{ steps.app.outputs.token }}
        with:
          use-label-strategy: true
          version-files: cargo
```

Things to know:

- Commits pushed with an App token start workflows again. The job's `if` above skips the action's own commit; without it, that run finds no PR and stops with a warning, but in the concurrency group it can cancel a release that is waiting.
- GitHub keeps one waiting run per concurrency group, so with three merges in quick succession the middle one's release can be cancelled. Re-run it.
- The release is sized by the pushed commit's PR, but built from the branch head when the run starts. If another PR merged in between, this release includes it, and that PR's own run releases again on top.
- `cargo`: workspace dependencies that pin the workspace's own crates with a `version` (as for publishing to crates.io) are not updated.

This action uses [release-on-merge-action-source](https://github.com/dexwritescode/release-on-merge-action-source)

## License

This Github Action is licensed under <a href="LICENSE">MIT license</a>.
