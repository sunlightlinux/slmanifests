---
name: Bug Report
about: Report a problem syncing or resolving the Sunlight Linux manifest
title: "[BUG] "
labels: bug
assignees: ''
---

## Description

A clear description of the problem (e.g. a project fails to sync, lands at the
wrong path, or the manifest won't resolve).

## Command

The exact `repo` command(s) you ran:

```
repo init -u https://github.com/sunlightlinux/slmanifests.git -b main
repo sync -j4
```

## Expected Behavior

What you expected to happen.

## Actual Behavior

What actually happened.

## Output

```
Paste the full repo / git output here
```

## Environment

- repo version: (`repo --version`)
- git version: (`git --version`)
- OS:
- Manifest revision: branch `main` / release snapshot `release-<tag>.xml`

## Additional Context

Which project(s) are affected, and any other detail that helps diagnose it.
