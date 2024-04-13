# Release Actions

This set of reusable GitHub Actions implements a versioned release workflow with the following actions:

- `rainstormy-actions/release/pr` creates a pull request for the release.
- `rainstormy-actions/release/tag` creates a release tag in Git.
- `rainstormy-actions/release/github` creates a draft GitHub release.
- `rainstormy-actions/release/npm` publishes a package to the npm registry.

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

#### Publish a package to npm
Use `rainstormy-actions/release/npm` to publish a package to the npm registry from a particular point in the Git history,
e.g. the release tag created by `rainstormy-actions/release/tag`.

It detects the appropriate package manager automatically, but expects you to set it up in advance.

```yaml
# .github/workflows/release-to-npm.yml
on:
  push:
    tags:
      - v*

jobs:
  publish-npm-package:
    runs-on: ubuntu-22.04
    permissions:
      contents: read # Allow the job to check out the repository.
      id-token: write # Allow npm to publish the package with provenance.
    steps:
      - name: Check out the repository
        uses: actions/checkout@v4
      - name: Install Node.js and npm
        uses: actions/setup-node@v4
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
        uses: rainstormy-actions/release/npm@v1
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

| Option                        | Description                                                                                                                                                     |
|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `access-level`                | The public visibility of the package; `public` or `restricted`.                                                                                                 |
| `npm-auth-token`              | A granular access token for the npm registry with `read` and `write` permissions for the package or the scope in which the package resides.                     |
| `packagejson-excluded-fields` | _(optional)_ A multiline string of property names to exclude from the published `package.json` file. Default: `packageManager`&#9166;`scripts`                  |
| `packagejson-path`            | _(optional)_ The location of the `package.json` file that declares the package to be published. Default value: `./package.json`.                                |
| `prerelease-tag`              | _(optional)_ The tag name to associate with a prerelease of the package in addition to the package version number. Default value: `next`.                       |
| `release-tag`                 | _(optional)_ The tag name to associate with a major, minor, or patch release of the package in addition to the package version number. Default value: `latest`. |
