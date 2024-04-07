# Release Actions

This set of reusable GitHub Actions implements a versioned release workflow with the following actions:

- `rainstormy-actions/release/pr` creates a pull request for the release.
- `rainstormy-actions/release/tag` creates a release tag in Git.
- `rainstormy-actions/release/github` creates a draft GitHub release.

### Usage
#### Create a pull request for the release
Use `rainstormy-actions/release/pr` to create a release branch and a corresponding pull request
that updates release artifacts, package definitions, changelogs etc. to the given semantic version number.

When triggered manually through a `workflow_dispatch`, you can provide the version number through an input parameter.
Alternatively, you can compute the version number as a step in the workflow.

The naming convention for the release branch is `release/<version>`.
If necessary, you can amend the commit on the release branch manually before merging the pull request.

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
  create-release-pr:
    runs-on: ubuntu-22.04
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
      - name: Create a release pull request
        uses: rainstormy-actions/release/pr@v1
        with:
          gh-auth-token: ${{ secrets.GH_AUTH_TOKEN }}
          version: ${{ inputs.version }}
```

| Option          | Description                                                                                                         |
|-----------------|---------------------------------------------------------------------------------------------------------------------|
| `gh-auth-token` | An authentication token for the GitHub CLI with scopes for `repo` and `read:org` in order to create a pull request. |
| `version`       | A string that contains a semantic version number on the form `<major>.<minor>.<patch>[-prerelease][+buildinfo]`.    |

#### Create a release tag
Use `rainstormy-actions/release/tag` to create a Git tag that points to a release commit,
e.g. the merge commit of the pull request created by `rainstormy-actions/release/pr`.

The naming convention for the release tag is `v<version>`.

```yaml
# .github/workflows/release-tag.yml
on:
  pull_request:
    branches:
      - main
    types:
      - closed

jobs:
  create-release-tag:
    if: github.event.pull_request.merged == true && startsWith(github.head_ref, 'release/')
    runs-on: ubuntu-22.04
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4
        with:
          # Use a separate access token to allow the push tag event in this workflow to trigger subsequent workflows, e.g. to create a GitHub release and to publish an npm package.
          # https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow
          token: ${{ secrets.GH_AUTH_TOKEN }}
      - name: Create a release tag
        uses: rainstormy-actions/release/tag@v1
        with:
          version: ${{ github.head_ref }}
```

| Option    | Description                                                                                                      |
|-----------|------------------------------------------------------------------------------------------------------------------|
| `version` | A string that contains a semantic version number on the form `<major>.<minor>.<patch>[-prerelease][+buildinfo]`. |

#### Create a draft GitHub release
Use `rainstormy-actions/release/github` to create a draft GitHub release that points to a release tag in Git,
e.g. the one created by `rainstormy-actions/release/tag`.

The expected naming convention for the release tag is `v<version>`.

```yaml
# .github/workflows/release-to-github.yml
on:
  push:
    tags:
      - v*

jobs:
  create-github-release:
    runs-on: ubuntu-22.04
    steps:
      - name: Create a draft GitHub release
        uses: rainstormy-actions/release/github@v1
        with:
          gh-auth-token: ${{ secrets.GH_AUTH_TOKEN }}
          version: ${{ github.ref_name }}
```

| Option          | Description                                                                                                         |
|-----------------|---------------------------------------------------------------------------------------------------------------------|
| `gh-auth-token` | An authentication token for the GitHub CLI with scopes for `repo` and `read:org` in order to create a pull request. |
| `version`       | A string that contains a semantic version number on the form `<major>.<minor>.<patch>[-prerelease][+buildinfo]`.    |
