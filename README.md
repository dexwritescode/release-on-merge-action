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
      - uses: dexwritescode/release-on-merge-action@v0.2.2
```

The only permissions needed for `GITHUB_TOKEN` to create tags and releases is `contents: write`.

List of inputs:

```yaml
  version-increment-strategy:  # Valid values: major|minor|patch|norelease
    description: 'The version number to increment. Options major|minor|patch|norelease'
    required: false
    default: 'patch'
  initial-version:
    description: 'The very first release version to create. The default Github tag will be v0.1.0'
    required: false
    default: '0.1.0'
  tag-prefix:
    description: 'Git tag prefix. Example the v in v1.2.3'
    required: false
    default: 'v'
  body:
    description: 'Body text to prepend the auto generated body'
    required: false
    default: ''
  generate-release-notes:
    description: 'Whether to generate release notes. Default true.'
    required: false
    default: true
  dry-run:
    description: 'Do not create a release, just log the oputput.'
    required: false
    default: false
```

This action uses [release-on-merge-action-source](https://github.com/dexwritescode/release-on-merge-action-source)
