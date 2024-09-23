# `rainstormy/release/major-tag`

Use the `rainstormy/release/major-tag` action to create a major-only version tag
in Git that points to a release commit, e.g. the merge commit of the pull
request created by the [`rainstormy/release/pr`](../pr/README.md) action.

The naming convention for the tag is `v<major>`.

It is complemented by the [`rainstormy/release/tag`](../tag/README.md) action,
which creates a corresponding full semantic version tag in Git.

> [!IMPORTANT]  
> As the action creates and pushes a Git tag, it requires a separate access
> token with permission to push tags to the Git repository, e.g. provided in
> the `token` parameter of
> the [`actions/checkout`](https://github.com/actions/checkout) action.

> [!CAUTION]  
> This action overwrites an existing major-only version tag in the Git. While
> the official Git documentation
> [strongly discourages](https://git-scm.com/docs/git-tag#_on_re_tagging)
> this approach, it is often observed in practice among reusable GitHub Actions.

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
      # - name: Create a full semantic version tag in Git
      #   uses: rainstormy/release/tag@v1
      #   with:
      #     version: ${{ github.head_ref }}
      - name: Create a major-only version tag in Git
        uses: rainstormy/release/major-tag@v1
        with:
          version: ${{ github.head_ref }}
```

## Options
### `version`
A string that contains a semantic version number on the
form `<major.minor.patch[-prerelease][+buildinfo]>`.
