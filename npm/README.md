# `rainstormy/release/npm`

Use the `rainstormy/release/npm` action to publish a package to an npm registry
from a particular point in the Git history, e.g. the full semantic version tag
created by the [`rainstormy/release/tag`](../tag/README.md) action.

It supports npm 10, pnpm 9, and Yarn 4 and detects the package manager
automatically from the package lockfile in the Git repository or from
the `packageManager` field in the `package.json` file.

> [!IMPORTANT]  
> As the action publishes to an npm registry using npm, pnpm, or Yarn, it
> requires Node.js, the npm registry, and the appropriate package manager to be
> set up prior to running the action, e.g. using
> the [`actions/setup-node`](https://github.com/actions/setup-node) action.

```yaml
# .github/workflows/release-npm.yml
on:
  push:
    tags:
      - v*

jobs:
  npm-package:
    runs-on: ubuntu-24.04
    timeout-minutes: 1
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
        uses: rainstormy/release/npm@v1
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

## Options
### `access-level`
The public visibility of the package; `public` or `restricted`.

### `npm-auth-token`
A granular access token for the npm registry with `read` and `write` permissions
for the package or the scope in which the package resides.

### `packagejson-excluded-fields` (optional)
A multiline string of property names to exclude from the
published `package.json` file.

Default value: `packageManager`&#9166;`scripts`

### `packagejson-path` (optional)
The location of the `package.json` file that declares the package to be
published.

Default value: `./package.json`.

### `prerelease-tag` (optional)
The tag name to associate with a prerelease of the package in addition to the
package version number.

Default value: `next`.

### `release-tag` (optional)
The tag name to associate with a major, minor, or patch release of the package
in addition to the package version number.

Default value: `latest`.
