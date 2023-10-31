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

**Optional** If no previous release found in the repo, the default configuration will create release and tag `v0.1.0`.

The below override will create release and tag `v1.2.3`
```yml
initial-version: '1.2.3'
```

### `tag-prefix`

**Optional** Git tag prefix. Example the v in `v1.2.3`. Default `v`.

```yml
tag-prefix: v
```

### `body`

**Optional** Body text to prepend the auto generated body.

```yml
body: 'Body text...'
```

### `generate-release-notes`

**Optional** Whether to auto-generate Github release notes. Default `true`.

```yml
generate-release-notes: true
```

### `dry-run`

**Optional** Do not create a release, just log the oputput if enabled. Default `false`.

```yml
dry-run: true
```

This action uses [release-on-merge-action-source](https://github.com/dexwritescode/release-on-merge-action-source)

## License

This Github Action is licensed under <a href="LICENSE">MIT license</a>.
