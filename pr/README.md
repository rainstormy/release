# `rainstormy/release/pr`

Use the `rainstormy/release/pr` action to create a release branch and a
corresponding pull request in GitHub that updates release artifacts, package
definitions, changelogs etc. to the given semantic version number.

When triggered manually through a `workflow_dispatch`, you can provide the
version number through an input parameter. Alternatively, you can compute the
version number as a step in the workflow.

The naming convention for the release branch is `release/<version>`. If
necessary, you can amend the commit on the release branch manually before
merging the pull request.

> [!IMPORTANT]  
> As the action creates and pushes a release commit to the release branch, it
> requires a separate access token with permission to push commits to the Git
> repository, e.g. provided in the `token` parameter of
> the [`actions/checkout`](https://github.com/actions/checkout) action.

```yaml
# .github/workflows/release.yml
on:
  workflow_dispatch:
    inputs:
      version:
        description: The semantic version number of the new release.
        type: string
        required: true

jobs:
  pull-request:
    runs-on: ubuntu-24.04
    timeout-minutes: 1
    permissions: { }
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4
        with:
          # Use a separate access token with permission to commit and push.
          token: ${{ secrets.GH_AUTH_TOKEN }}
      #
      # Apply your own steps here to generate release artifacts and update version numbers.
      #
      # - name: Prepare the release
      #   run: npm run release $VERSION
      #   env:
      #     VERSION: ${{ inputs.version }}
      #
      - name: Create a release-triggering pull request in GitHub
        uses: rainstormy/release/pr@v1
        with:
          gh-auth-token: ${{ secrets.GH_AUTH_TOKEN }}
          version: ${{ inputs.version }}
```

## Options
### `gh-auth-token`
An access token for GitHub with scopes for `repo` and `read:org` in order to
create a pull request.

### `version`
A string that contains a semantic version number on the
form `<major.minor.patch[-prerelease][+buildinfo]>`.
