# slmanifests

The [`repo`](https://gerrit.googlesource.com/git-repo) tool manifest for **Sunlight Linux** — a UEFI-only Linux distribution. This repository defines which Sunlight Linux source repositories make up a full checkout and at which revision they track.

It contains a single source of truth: [`default.xml`](default.xml).

## Quick Start

Install `repo` (Google's multi-repository tool), then bootstrap a full Sunlight Linux tree:

```bash
repo init -u https://github.com/sunlightlinux/slmanifests.git -b main
repo sync -j4
```

`repo init` clones this manifest; `repo sync` clones (or updates) every project it lists.

## Projects

| Project | Path | Purpose |
|---------|------|---------|
| `slkernel`    | `kernel`        | Kernel packaging / config |
| `slpkgs`      | `pkgs`          | `void-packages` fork, rebranded to Sunlight Linux |
| `slinit`      | `src/init`      | Init system / service manager (PID 1), written in Go |
| `slinstaller` | `src/installer` | System installer |
| `slmanifests` | `manifests`     | This manifest repo, checked out into the tree |

After `repo sync` the tree looks like:

```
.
├── kernel/          # slkernel
├── pkgs/            # slpkgs
├── manifests/       # slmanifests (this repo)
└── src/
    ├── init/        # slinit
    └── installer/   # slinstaller
```

## Branch Model

Every project tracks **`main`** (no pinned SHAs). The remote `slgh` points at `https://github.com/sunlightlinux/`, and `repo sync` defaults to 4 parallel jobs.

## Releases

For a reproducible, auditable build, snapshot the current state of every project into a pinned manifest:

```bash
repo manifest -r -o release-<tag>.xml
```

The resulting file records an exact SHA per project, so

```bash
repo init -u https://github.com/sunlightlinux/slmanifests.git -m release-<tag>.xml
repo sync
```

reproduces that exact tree.

## Useful Commands

| Command | What it does |
|---------|--------------|
| `repo sync -j4`                | Clone/update all projects |
| `repo manifest`                | Print the fully resolved manifest |
| `repo manifest -r -o snap.xml` | Snapshot pinned SHAs into `snap.xml` |
| `repo forall -c 'git status'`  | Run a command in every project |
| `repo status`                  | Show working-tree status across projects |

## Documentation

- [CONTRIBUTING.md](CONTRIBUTING.md) — how to propose manifest changes
- [SECURITY.md](SECURITY.md) — reporting security / supply-chain issues
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) — community standards
- [CLAUDE.md](CLAUDE.md) / [EXAMPLES.md](EXAMPLES.md) — guidelines for LLM-assisted edits to this manifest

## License

[Apache License 2.0](LICENSE).
