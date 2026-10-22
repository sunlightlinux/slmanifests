# Contributing to slmanifests

Thank you for your interest in contributing to slmanifests, the [`repo`](https://gerrit.googlesource.com/git-repo) manifest for Sunlight Linux!

This repository contains no code — just `default.xml`, the manifest that tells Google's `repo` tool which Sunlight Linux repositories to check out. Contributions are almost always edits to that one file.

## How to Contribute

### Reporting Issues

- Use the GitHub issue tracker to report problems.
- Include the `repo` command you ran, the full output, and your `repo --version`.
- For a sync failure, include the offending `<project>` line.

### Submitting Changes

1. Fork the repository
2. Create a feature branch (`git checkout -b manifest/add-foo`)
3. Edit `default.xml`
4. Validate it is well-formed: `xmllint --noout default.xml`
5. Confirm `repo` can resolve it (see Testing)
6. Commit with a clear message
7. Push to your fork and open a Pull Request

### Manifest Conventions

- **All projects track `main`.** Do not pin a SHA or tag on a `<project>` in `default.xml`; pinned revisions belong in release snapshots, not here.
- **One remote (`slgh`).** Only add a second `<remote>` if a project genuinely lives outside `github.com/sunlightlinux`.
- **`name` is the repo, `path` is the checkout location.** Keep `path` stable — changing it relocates every existing checkout and breaks CI.
- **Keep the scope deliberate.** The manifest lists exactly seven projects on purpose (`linux-sunlight` and CI repos are excluded). Adding one is a real decision; explain why in the PR.
- **Match the existing style:** two-space indent, `name` before `path`, one comment per project.
- **Keep the header usage comment current** if you change how the manifest is used.

### Testing

- **Well-formedness:** `xmllint --noout default.xml`
- **Resolution:** in a scratch directory,
  ```bash
  repo init -u <your-fork-url> -b <your-branch>
  repo manifest
  ```
  prints the fully resolved manifest without syncing.
- **Full sync (optional, heavy):** `repo sync -j4` actually clones every project at the manifest revision.

### Release Snapshots

A reproducible release is a pinned copy of this manifest:

```bash
repo manifest -r -o release-<tag>.xml
```

This records the current SHA of every project. Commit such snapshots as `release-<tag>.xml` files — never as edits to `default.xml`, which must keep tracking `main`.

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.
