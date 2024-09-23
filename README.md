# Release Actions

This set of reusable GitHub Actions implements a versioned release workflow with
the following actions:

- [`rainstormy/release/pr`](./pr/README.md) creates a release-triggering pull
  request in GitHub.
- [`rainstormy/release/tag`](./tag/README.md) creates a full semantic version
  tag in Git.
- [`rainstormy/release/major-tag`](./major-tag/README.md) creates a major-only
  version tag in Git.
- [`rainstormy/release/github`](./github/README.md) creates a draft GitHub
  release.
- [`rainstormy/release/npm`](./npm/README.md) publishes a package to an npm
  registry.
