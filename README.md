# Release Actions

This set of reusable GitHub Actions implements a versioned release workflow with
the following actions:

- [`rainstormy-actions/release/pr`](./pr/README.md) creates a release-triggering
  pull request in GitHub.
- [`rainstormy-actions/release/tag`](./tag/README.md) creates a full semantic
  version tag in Git.
- [`rainstormy-actions/release/major-tag`](./major-tag/README.md) creates a
  major-only version tag in Git.
- [`rainstormy-actions/release/github`](./github/README.md) creates a draft
  GitHub release.
- [`rainstormy-actions/release/npm`](./npm/README.md) publishes a package to an
  npm registry.
