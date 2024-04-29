# Release Actions

This set of reusable GitHub Actions implements a versioned release workflow with
the following actions:

- `rainstormy-actions/release/pr` creates a release-triggering pull request.
- `rainstormy-actions/release/tag` creates a release tag in Git.
- `rainstormy-actions/release/github` creates a draft GitHub release.
- `rainstormy-actions/release/npm` publishes a package to an npm registry.

## Create a release-triggering pull request
Use `rainstormy-actions/release/pr` to create a release branch and a
corresponding pull request that updates release artifacts, package definitions,
changelogs etc. to the given semantic version number.

When triggered manually through a `workflow_dispatch`, you can provide the
version number through an input parameter. Alternatively, you can compute the
version number as a step in the workflow.

The naming convention for the release branch is `release/<version>`. If
necessary, you can amend the commit on the release branch manually before
merging the pull request.

> [!IMPORTANT]  
> As the action creates and pushes a release commit to the release branch, it
> requires a separate access token with permission to push commits to the Git
> repository, e.g. provided in the `token` parameter
> of [actions/checkout](https://github.com/actions/checkout).

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
    runs-on: ubuntu-22.04
    timeout-minutes: 1
    permissions: { }
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4 # https://github.com/actions/checkout
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
        uses: rainstormy-actions/release/pr@v1 # https://github.com/rainstormy-actions/release
        with:
          gh-auth-token: ${{ secrets.GH_AUTH_TOKEN }}
          version: ${{ inputs.version }}
```

### Options
#### `gh-auth-token`
An access token for GitHub with scopes for `repo` and `read:org` in order to
create a pull request.

#### `version`
A string that contains a semantic version number on the
form `<major>.<minor>.<patch>[-prerelease][+buildinfo]`.

## Create a release tag
Use `rainstormy-actions/release/tag` to create a Git tag that points to a
release commit, e.g. the merge commit of the pull request created
by `rainstormy-actions/release/pr`.

The naming convention for the release tag is `v<version>`.

> [!IMPORTANT]  
> As the action creates and pushes a release tag, it requires a separate access
> token with permission to push tags to the Git repository, e.g. provided in
> the `token` parameter
> of [actions/checkout](https://github.com/actions/checkout).

```yaml
# .github/workflows/release-tag.yml
on:
  pull_request:
    branches:
      - main
    types:
      - closed

jobs:
  tag:
    if: github.event.pull_request.merged == true && startsWith(github.head_ref, 'release/')
    runs-on: ubuntu-22.04
    timeout-minutes: 1
    permissions: { }
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4 # https://github.com/actions/checkout
        with:
          # Use a separate access token to allow the push tag event in this workflow to trigger subsequent workflows, e.g. to create a GitHub release and to publish an npm package.
          # https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow
          token: ${{ secrets.GH_AUTH_TOKEN }}
      - name: Create a release tag
        uses: rainstormy-actions/release/tag@v1 # https://github.com/rainstormy-actions/release
        with:
          version: ${{ github.head_ref }}
```

### Options
#### `version`
A string that contains a semantic version number on the
form `<major>.<minor>.<patch>[-prerelease][+buildinfo]`.

## Create a draft GitHub release
Use `rainstormy-actions/release/github` to create a draft GitHub release that
points to a release tag in Git, e.g. the one created
by `rainstormy-actions/release/tag`.

The expected naming convention for the release tag is `v<version>`.

> [!IMPORTANT]  
> As the action creates a GitHub release from a Git tag, it requires the Git
> repository to be checked out prior to running the action, e.g.
> using [actions/checkout](https://github.com/actions/checkout).

```yaml
# .github/workflows/release-github.yml
on:
  push:
    tags:
      - v*

jobs:
  github-release:
    runs-on: ubuntu-22.04
    timeout-minutes: 1
    permissions:
      contents: read # Allow the job to check out the repository.
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4 # https://github.com/actions/checkout
      - name: Create a draft GitHub release
        uses: rainstormy-actions/release/github@v1 # https://github.com/rainstormy-actions/release
        with:
          gh-auth-token: ${{ secrets.GH_AUTH_TOKEN }}
          version: ${{ github.ref_name }}
```

### Options
#### `gh-auth-token`
An access token for GitHub with scopes for `repo` and `read:org` in order to
create a GitHub release.

#### `version`
A string that contains a semantic version number on the
form `<major>.<minor>.<patch>[-prerelease][+buildinfo]`.

## Publish a package to npm
Use `rainstormy-actions/release/npm` to publish a package to an npm registry
from a particular point in the Git history, e.g. the release tag created
by `rainstormy-actions/release/tag`.

It supports npm 10, pnpm 9, and Yarn 4 and detects the package manager
automatically from the package lockfile in the Git repository or from
the `packageManager` field in the `package.json` file.

> [!IMPORTANT]  
> As the action publishes to an npm registry using npm, pnpm, or Yarn, it
> requires Node.js, the npm registry, and the appropriate package manager to be
> set up prior to running the action, e.g.
> using [actions/setup-node](https://github.com/actions/setup-node).

```yaml
# .github/workflows/release-npm.yml
on:
  push:
    tags:
      - v*

jobs:
  npm-package:
    runs-on: ubuntu-22.04
    timeout-minutes: 1
    permissions:
      contents: read # Allow the job to check out the repository.
      id-token: write # Allow npm to publish the package with provenance.
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4 # https://github.com/actions/checkout
      - name: Install Node.js and npm
        uses: actions/setup-node@v4 # https://github.com/actions/setup-node
        with:
          node-version: 20
          registry-url: https://registry.npmjs.org
      #
      # Apply your own steps here to generate build artifacts and prepare the package for publishing.
      #
      # - name: Prepare the package
      #   run: npm run build
      #
      - name: Publish a package to npm
        uses: rainstormy-actions/release/npm@v1 # https://github.com/rainstormy-actions/release
        with:
          access-level: public
          npm-auth-token: ${{ secrets.NPM_AUTH_TOKEN }}
          # packagejson-excluded-fields: |
          #   packageManager
          #   scripts
          # packagejson-path: ./package.json
          # prerelease-tag: next
          # release-tag: latest
```

### Options
#### `access-level`
The public visibility of the package; `public` or `restricted`.

#### `npm-auth-token`
A granular access token for the npm registry with `read` and `write` permissions
for the package or the scope in which the package resides.

#### `packagejson-excluded-fields` (optional)
A multiline string of property names to exclude from the
published `package.json` file.

Default value: `packageManager`&#9166;`scripts`

#### `packagejson-path` (optional)
The location of the `package.json` file that declares the package to be
published.

Default value: `./package.json`.

#### `prerelease-tag` (optional)
The tag name to associate with a prerelease of the package in addition to the
package version number.

Default value: `next`.

#### `release-tag` (optional)
The tag name to associate with a major, minor, or patch release of the package
in addition to the package version number.

Default value: `latest`.
