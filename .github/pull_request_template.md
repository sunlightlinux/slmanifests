## Description

Brief description of the manifest change.

## Type of Change

- [ ] Add a project
- [ ] Remove a project
- [ ] Re-point a project (name / path / revision)
- [ ] Release snapshot (`release-<tag>.xml`)
- [ ] Documentation update

## Validation

- [ ] `xmllint --noout default.xml` passes
- [ ] `repo manifest` resolves cleanly
- [ ] `repo sync -j4` succeeds (if path / remote / name changed)

## Checklist

- [ ] Change is surgical — only the lines the request needs
- [ ] `default.xml` still tracks `main` (no pinned SHAs outside a release snapshot)
- [ ] Any project-scope change is justified above
- [ ] No new remote / host introduced without reason (supply-chain)
