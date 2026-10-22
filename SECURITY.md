# Security Policy

## Scope of this repository

`slmanifests` contains no executable code — only the [`repo`](https://gerrit.googlesource.com/git-repo) tool manifest (`default.xml`) that defines which Sunlight Linux repositories are checked out and at which revision they track. There is nothing to run here, but the manifest is **supply-chain sensitive**: it decides what source every developer and CI pipeline fetches and builds.

## Reporting a Vulnerability

If you discover a security issue affecting Sunlight Linux's source distribution — for example a manifest pointing at an unexpected remote, a tampered release snapshot, or a hijacked project name — please report it responsibly.

**Do NOT open a public GitHub issue for security vulnerabilities.**

Instead, please send an email to: ionut_n2001@yahoo.com

Include:
- Description of the issue
- The manifest revision (or release snapshot) affected
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

You should receive a response within 48 hours. We will work with you to understand the issue and coordinate a fix before any public disclosure.

## Supply-chain considerations

- **Remotes are HTTPS only.** The `slgh` remote fetches over `https://github.com/sunlightlinux/`. A manifest change that introduces an `http://` remote, an unknown host, or an unexpected org should be treated as suspicious.
- **`main` tracks heads; releases are pinned.** Day to day the manifest follows `main` for every project. Reproducible, auditable builds come from pinned snapshots produced with `repo manifest -r -o release-<tag>.xml`. Verify a release snapshot's SHAs against the upstream repositories before trusting it.
- **Project scope is intentional.** The manifest lists exactly seven projects. An unexplained new `<project>`, or a changed `name`/`path`, widens what gets built and shipped — review such changes as you would a new dependency.
