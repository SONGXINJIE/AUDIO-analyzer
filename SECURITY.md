# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| `0.1.0` (current pre-release) | Yes |
| Any earlier version | No |

This project is pre-1.0. Security fixes are applied to `main` and included in the
next tagged release; no long-term maintenance branches exist yet.

## Reporting a Vulnerability

**Do not open a public Issue for security vulnerabilities.**

Use the repository's **Security Advisories → Report a vulnerability** feature on
GitHub. It creates a private report visible only to maintainers and supports
private discussion and coordinated disclosure. If that feature is unavailable,
contact a maintainer privately through GitHub rather than posting publicly.

Please include:

- the affected version or commit;
- a description of the issue and its potential impact;
- reproduction steps, a minimal proof of concept, or a failing test;
- any suggested mitigation, if you have one.

## Response Expectations

| Stage | Target |
|---|---|
| acknowledgement | within 5 working days |
| initial assessment | within 10 working days |
| fix or documented mitigation | in the next release, severity-dependent |

Reporters who follow this process will be credited in the release notes unless
they prefer to remain anonymous.

## Scope and Known Boundaries

The following are explicitly out of scope for security reports:

- issues that require the attacker to already have local write access to the
  repository or virtual environment;
- vulnerabilities in third-party dependencies — report those upstream, then open
  a normal Issue here so we can track the upgrade;
- denial of service caused by analysing extremely large or malformed audio files
  beyond the documented size and duration limits.

Design-level security constraints are recorded in the
[design specification](docs/superpowers/specs/2026-11-14-audio-analyzer-platform-design.md):
profile name whitelist, path traversal rejection, 200-character error messages,
cache serialisation via JSON with explicit schema validation (no pickle), and
rejection of non-finite numeric results.
