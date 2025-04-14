## `rainstormy/release/github`

Use the `rainstormy/release/github` action to create a draft GitHub release that
points to a full semantic version tag in Git, e.g. the one created by
the [`rainstormy/release/tag`](../tag/README.md) action.

It may extract release notes from a Markdown changelog file
in [Keep a Changelog](https://keepachangelog.com/en/1.1.0) format.

The expected naming convention for the tag
is `v<major.minor.patch[-prerelease][+buildinfo]>`.

> [!IMPORTANT]  
> As the action creates a GitHub release from a Git tag, it requires the Git
> repository to be checked out prior to running the action, e.g. using
> the [`actions/checkout`](https://github.com/actions/checkout) action.

```yaml
# .github/workflows/release-github.yml
on:
  push:
    tags:
      - v*

jobs:
  github-release:
    runs-on: ubuntu-24.04
    timeout-minutes: 1
    permissions:
      contents: read # Allow the job to check out the repository.
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4
      - name: Create a draft GitHub release
        uses: rainstormy/release/github@v1
        with:
          # changelog-path: ./CHANGELOG.md
          gh-auth-token: ${{ secrets.GH_AUTH_TOKEN }}
          version: ${{ github.ref_name }}
```

## Options
### `changelog-path`
The location of a Markdown changelog file that contains the release notes
in [Keep a Changelog](https://keepachangelog.com/en/1.1.0) format.

### `gh-auth-token`
An access token for GitHub with scopes for `repo` and `read:org` in order to
create a GitHub release.

### `version`
A string that contains a semantic version number on the
form `<major.minor.patch[-prerelease][+buildinfo]>`.
