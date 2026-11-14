# Changelog

All notable changes to this project are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

No tagged release exists yet. The first tag will be created when the implementation
reaches its first usable milestone; until then, all changes accumulate under
`Unreleased`.

## [Unreleased]

### Added

- MIT license, contribution guide, code of conduct, security policy and changelog.
- Git workflow: GitHub Flow, Conventional Commits, pre-commit configuration,
  GitHub Actions CI, Conventional Commits title check and Dependabot.
- Design specification for the plugin-based, CPU-only audio analysis platform.
- Minimal importable package skeleton exposing `audio_analyzer.__version__`.

### Changed

- Version reset from the legacy `5.8.0` line to `0.1.0`, reflecting the fresh start.
- README and CONTEXT rewritten to match the CPU-only plugin architecture;
  obsolete CUDA / CuPy / GPU memory guidance removed.
- Design specification paths generalised from absolute local paths to repository-relative wording.

### Removed

- Placeholder repository URLs and the dangling console-script entry point.

### Compliance

- `audio-test/` (commercial music) and `.workbuddy/` (internal notes) excluded
  from version control. Only the two synthetic WAV fixtures are tracked.
