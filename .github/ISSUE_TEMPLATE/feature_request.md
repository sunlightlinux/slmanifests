---
name: Manifest Change Request
about: Propose adding, removing, or re-pointing a project in the manifest
title: "[MANIFEST] "
labels: enhancement
assignees: ''
---

## Description

What change to `default.xml` are you proposing?

## Project Details

For a new or changed project:

- Repo name (under `github.com/sunlightlinux/`):
- Checkout `path`:
- Revision: `main` (default) / other:
- Optional? (should it carry `groups="notdefault"`?):

## Rationale

Why does this belong in the manifest? The manifest deliberately tracks a small,
fixed set of projects — explain what this adds.

## Impact

Does this change an existing `name`/`path` (which relocates checkouts and may
break CI)? Note any consumers affected.

## Alternatives Considered

e.g. keeping it out of the default manifest, a separate `<include>`, or a
release snapshot.
