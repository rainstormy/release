# `rainstormy-actions/release/tag`

Use the `rainstormy-actions/release/tag` action to create a full semantic
version tag in Git that points to a release commit, e.g. the merge commit of the
pull request created by the [`rainstormy-actions/release/pr`](../pr/README.md)
action.

The naming convention for the tag
is `v<major.minor.patch[-prerelease][+buildinfo]>`.

It is complemented by
the [`rainstormy-actions/release/major-tag`](../major-tag/README.md) action,
which creates a corresponding major-only version tag in Git.

> [!IMPORTANT]  
> As the action creates and pushes a Git tag, it requires a separate access
> token with permission to push tags to the Git repository, e.g. provided in
> the `token` parameter
> of [actions/checkout](https://github.com/actions/checkout).

> [!IMPORTANT]  
> This action fails if the full semantic version tag already exists in the Git
> repository.

```yaml
# .github/workflows/release-tags.yml
on:
  pull_request:
    branches:
      - main
    types:
      - closed

jobs:
  tags:
    if: github.event.pull_request.merged == true && startsWith(github.head_ref, 'release/')
    runs-on: ubuntu-24.04
    timeout-minutes: 1
    permissions: { }
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4
        with:
          # Use a separate access token to allow the push tag event in this workflow to trigger subsequent workflows, e.g. to create a GitHub release and to publish an npm package.
          # https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow
          token: ${{ secrets.GH_AUTH_TOKEN }}
      - name: Create a full semantic version tag in Git
        uses: rainstormy-actions/release/tag@v1
        with:
          version: ${{ github.head_ref }}
      # - name: Create a major-only version tag in Git
      #   uses: rainstormy-actions/release/major-tag@v1
      #   with:
      #     version: ${{ github.head_ref }}
```

## Options
### `version`
A string that contains a semantic version number on the
form `<major.minor.patch[-prerelease][+buildinfo]>`.
