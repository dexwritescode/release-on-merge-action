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

This action uses [release-on-merge-action-source](https://github.com/dexwritescode/release-on-merge-action-source)

## License

This Github Action is licensed under <a href="LICENSE">MIT license</a>.
