# Changelog

This file documents all notable changes to this project.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0),
and this project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
### Changed
- Reformat extracted release notes in `rainstormy/release/github`:
  - Promote level 3 headings (`###`) to level 2 (`##`).
  - Undo line wrapping and indents in list items.

## [1.1.0] - 2024-08-11
### Added
- New action: `rainstormy/release/major-tag`.
- Extraction of release notes from Markdown changelogs
  in `rainstormy/release/github`.
- New input in `rainstormy/release/github`: `changelog-path`.

## [1.0.0] - 2024-05-12
### Added
- [MIT license](https://choosealicense.com/licenses/mit).
- New action: `rainstormy/release/pr`.
- New action: `rainstormy/release/tag`.
- New action: `rainstormy/release/github`.
- New action: `rainstormy/release/npm`.

[unreleased]: https://github.com/rainstormy/release/releases/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/rainstormy/release/releases/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/rainstormy/release/releases/tag/v1.0.0
