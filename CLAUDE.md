# CLAUDE.md

Behavioral guidelines for LLM-assisted work on **slmanifests** — the
[`repo`](https://gerrit.googlesource.com/git-repo) tool manifest for
Sunlight Linux. The entire repository is one file, `default.xml`, that
declares which Sunlight Linux repositories a full checkout contains.

> Derived from [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)
> (MIT), based on [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876)
> on LLM coding pitfalls. Adapted for slmanifests: `repo` manifest semantics,
> XML correctness, and supply-chain caution.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial
edits (fixing a typo in a comment) use judgment — not every change needs the
full rigor. But remember this file decides what every developer and CI
pipeline checks out, so "trivial" is rarer here than it looks.

See [EXAMPLES.md](EXAMPLES.md) for concrete manifest examples.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before editing the manifest:
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
  The manifest format has many near-synonyms:
  - `name` (the repo on the remote) vs `path` (where it lands in the tree).
  - `revision` (what to check out) vs `upstream` (the branch a pinned SHA
    came from) vs `dest-branch` (the branch a checkout starts on).
  - editing a `<project>` vs `<extend-project>` vs `<remove-project>` +
    re-adding it.
  - pinning a SHA/tag vs tracking `main`.
  Surface which is meant.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.
- **Check `repo`'s manifest format first.** For anything that looks like a
  feature request, check whether the manifest format already has the concept
  (`<include>`, `groups=`, `<copyfile>`, `<linkfile>`, `<extend-project>`,
  `<annotation>`) before inventing structure. Match its semantics.
- **Respect the deliberate scope.** The manifest lists exactly five projects;
  `linux-sunlight` and CI repos are excluded on purpose. Don't "helpfully"
  add them.

## 2. Simplicity First

**Minimum manifest that solves the problem. Nothing speculative.**

- No projects, remotes, or groups beyond what was asked.
- No `<include>` splitting for five projects — one `default.xml` is right.
- No second `<remote>` unless a project genuinely lives outside
  `github.com/sunlightlinux`.
- No `<copyfile>`/`<linkfile>` unless a real consumer needs the file in two
  places.
- **No pinned SHAs on `main`.** Tracking heads is the whole point of the
  default manifest; pinning belongs in release snapshots (`repo manifest -r`).
- One clear `<project>` line beats a premature `groups=`/`<extend-project>`
  scheme.

Ask yourself: "Would a maintainer say this manifest is overcomplicated?"
If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing the manifest:
- Don't reorder or reformat the whole file to add one project.
- Don't add `upstream=`/`dest-branch=`/`groups=` attributes "while you're in
  there."
- Match existing style: two-space indent, attribute order (`name` then
  `path`), one blank line + comment per project.
- Keep the header usage comment; update it only if the usage actually changed.
- Don't rename `path=` in passing — it relocates everyone's checkout. Renames
  should be their own, clearly-justified change.

**slmanifests-specific:**
- **Don't break `repo` consumers.** Changing a `name` or `path`, or removing a
  project, breaks existing checkouts and CI. That needs an explicit ask.
- **Don't pin `default.xml`.** If a reproducible build is wanted, produce a
  separate `release-<tag>.xml` snapshot — don't bake SHAs into the tracking
  manifest.

The test: every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add a project" → "default.xml is well-formed, `repo` resolves it, and the
  project appears at the intended path."
- "Fix the sync error" → "Reproduce the failure, identify the wrong
  attribute, confirm `repo manifest` resolves cleanly afterward."
- "Cut a release" → "`repo manifest -r` produces a snapshot whose SHAs match
  the upstream heads."

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```

**Verification commands in slmanifests:**
- `xmllint --noout default.xml` — XML well-formedness, catches typos fast.
- `repo manifest` — print the fully resolved manifest (validates structure
  and attributes without syncing).
- `repo init -u <url> -b <branch> && repo manifest` — resolve a branch/fork
  end to end in a scratch directory.
- `repo sync -j4` — full clone of every project (heavy; use when path/remote
  changes need real confirmation).
- `repo manifest -r -o release-<tag>.xml` — produce a pinned snapshot.

Strong success criteria let you loop independently. Weak criteria
("make it sync") require constant clarification.

---

## slmanifests Project Context

### Positioning
- The org-wide `repo` manifest for **Sunlight Linux** (UEFI-only distro).
- One source of truth: `default.xml`.
- Consumed by developers and CI via `repo init` + `repo sync`.

### Invariants
- **Five projects, on purpose:** `slkernel` (`kernel`), `slpkgs` (`pkgs`),
  `slinit` (`src/init`), `slinstaller` (`src/installer`), `slmanifests`
  (`manifests`). `linux-sunlight` and CI repos are intentionally excluded.
- **One remote:** `slgh` → `https://github.com/sunlightlinux/` (HTTPS only).
- **Everything tracks `main`** — no pinned SHAs in `default.xml`.
- **Releases are snapshots:** reproducibility comes from
  `repo manifest -r -o release-<tag>.xml`, not from editing the tracking
  manifest.

### Key file
- `default.xml` — the whole repository. A `<remote>`, a `<default>`, and five
  `<project>` entries.

### Reference sources
- **`repo` manifest format**:
  <https://gerrit.googlesource.com/git-repo/+/HEAD/docs/manifest-format.md> —
  authoritative grammar for every element/attribute. When in doubt about
  semantics, check it first.

### Commit style
- **Bundle, don't over-split.** One commit per session of work. Commit
  messages: `area: short summary` + a paragraph on *why*, not *what* (the diff
  already shows what).
