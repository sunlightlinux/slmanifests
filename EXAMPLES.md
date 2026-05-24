# EXAMPLES.md

Concrete manifest examples for each of the four principles in
[CLAUDE.md](CLAUDE.md), grounded in real slmanifests scenarios (the `repo`
tool, `default.xml`, release snapshots).

> Derived from [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)
> (MIT). Examples rewritten for `repo` manifests + slmanifests.

---

## 1. Think Before Coding

### Example 1 — Ambiguous Feature Request

**User request:** "Add the docs repo to the manifest."

**What LLMs often do (wrong assumption):**

```xml
<project name="docs" path="docs" />
```

**Problem:** several things are underspecified, and the manifest format has
near-synonyms that change behavior:

- Is the repo named `docs` or `sldocs`? (`name` is the *exact* repo under
  `github.com/sunlightlinux/`.)
- Where should it land — top-level `docs/`, or under `src/`? (`path`.)
- Does it track `main` like everything else, or a different branch?
  (`revision` overrides the default.)
- Is `docs` even in scope? The manifest deliberately lists five projects.

**What should happen (surface the choice):**

> The manifest currently tracks five projects on purpose. Before I add one:
>
> 1. Exact repo name under `github.com/sunlightlinux/` — `docs` or `sldocs`?
> 2. Checkout `path` — top-level `docs/`, or `src/docs/`?
> 3. Track `main` (the default), or a different branch via `revision`?
>
> Also — is this part of a *full* checkout, or optional? If optional, it
> should carry `groups="notdefault"` so a plain `repo sync` skips it.

### Example 2 — Multiple Interpretations

**User request:** "Make `repo sync` faster."

**What LLMs often do (pick silently):**

```xml
<default remote="slgh" revision="main" sync-j="16" />
```

**What should happen:**

> "Faster sync" could mean several things, with different blast radius:
>
> 1. **Raise `sync-j`** — more parallel project fetches. Helps with many
>    projects; we only have five, so the win is small and it can hammer the
>    server.
> 2. **Shallow clones** — `clone-depth="1"` per project (or `repo sync
>    --depth=1`) skips history. Big disk/time win, but breaks `git log` and
>    bisect in the checkout.
> 3. **Partial clone** — `repo init --partial-clone` fetches blobs on demand.
>    Best general-purpose win, but it's an `init`-time choice, not a manifest
>    attribute.
>
> Which trade-off do you want — and should it be baked into the manifest
> (affects everyone) or left to the developer's `repo` flags?

---

## 2. Simplicity First

### Example 1 — Over-abstraction

**User request:** "Organize the manifest so the userland repos are grouped
together."

**What LLMs often do (overengineered):**

```xml
<include name="remotes.xml" />
<include name="kernel.xml" />
<include name="userland.xml" />
<include name="tooling.xml" />
```

…with four extra files, each holding one or two `<project>` lines, plus
`groups="kernel"` / `groups="userland"` annotations everywhere.

**Problem:** there are five projects. Splitting them across four `<include>`
files and a group taxonomy adds indirection nobody can follow, for a tree that
fits on one screen.

**What should happen (minimum manifest):**

Keep one `default.xml`. If grouping is genuinely useful later (e.g. "sync only
userland"), add `groups=` *in place* when a second consumer actually needs it:

```xml
<project name="slpkgs"      path="pkgs"          groups="userland" />
<project name="slinstaller" path="src/installer" groups="userland" />
```

Five lines, one file. Add `<include>` splitting when the manifest outgrows a
screen, not before.

### Example 2 — Speculative Features

**User request:** "Make sure each project's license file is easy to find at
the tree root."

**What LLMs often do (feature creep):**

```xml
<project name="slinit" path="src/init">
  <copyfile src="LICENSE" dest="licenses/slinit-LICENSE" />
  <linkfile src="README.md" dest="docs/slinit-README.md" />
  <annotation name="tier" value="core" />
  <copyfile src="CHANGELOG.md" dest="changelogs/slinit.md" />
</project>
```

**Problem:** the ask was "find the license." That's one `<copyfile>`. The
links, annotation, and changelog copy are invented.

**What should happen:**

```xml
<project name="slinit" path="src/init">
  <copyfile src="LICENSE" dest="licenses/slinit-LICENSE" />
</project>
```

One `<copyfile>`. Add the rest when someone actually needs it.

---

## 3. Surgical Changes

### Example 1 — Drive-by "Improvements"

**User request:** "Add the installer project."

**What LLMs often do (too much):**

```diff
-  <default remote="slgh"
-           revision="main"
-           sync-j="4" />
+  <default remote="slgh"
+           revision="refs/heads/main"
+           sync-j="8"
+           clone-depth="1" />

-  <project name="slkernel" path="kernel" />
-  <project name="slpkgs" path="pkgs" />
-  <project name="slinit" path="src/init" />
+  <project name="slkernel"    path="kernel"        upstream="main" />
+  <project name="slinstaller" path="src/installer" upstream="main" />
+  <project name="slinit"      path="src/init"      upstream="main" />
+  <project name="slpkgs"      path="pkgs"          upstream="main" />
```

**Problems:**
- Changed `sync-j` and added `clone-depth` (not asked; changes everyone's sync
  behavior).
- Added `upstream=` to unrelated projects (not asked).
- Reordered and re-indented existing lines, burying the one real change.
- The actual task is **one new line.**

**What should happen (surgical):**

```diff
   <project name="slinit" path="src/init" />
+
+  <!-- System installer. -->
+  <project name="slinstaller" path="src/installer" />
```

One project, matching the existing comment-per-project style. Nothing else
moves.

### Example 2 — Style Drift

**User request:** "The installer should check out to `src/installer`, not
`installer` — fix the path."

**What LLMs often do (reformat everything):**

```diff
-<manifest>
-  <remote name="slgh" fetch="https://github.com/sunlightlinux/" />
-  <default remote="slgh" revision="main" sync-j="4" />
-  <project name="slinstaller" path="installer" />
-</manifest>
+<manifest>
+  <!-- Remotes -->
+  <remote name="slgh" fetch="https://github.com/sunlightlinux/" review="..." />
+
+  <!-- Defaults -->
+  <default remote="slgh" revision="main" sync-j="4" />
+
+  <!-- Projects -->
+  <project name="slinstaller" path="src/installer" dest-branch="main" />
+</manifest>
```

**Problems:**
- Added a `review=` remote attribute and a `dest-branch=` (not asked).
- Re-commented and re-spaced the whole file.
- The fix is **one attribute.**

**What should happen:**

```diff
-  <project name="slinstaller" path="installer" />
+  <project name="slinstaller" path="src/installer" />
```

One attribute. Nothing else changes.

---

## 4. Goal-Driven Execution

### Example 1 — Reproduce, Then Fix

**User request:** "`repo sync` fails for the kernel project — fix it."

**What LLMs often do (fix without reproducing):**

```diff
-  <project name="slkernel" path="kernel" />
+  <project name="kernel" path="kernel" revision="master" />
```

A guess that changes both the repo name and the revision — likely making it
worse.

**What should happen (reproduce-first):**

```
Plan:
1. Reproduce: run `repo manifest` and `xmllint --noout default.xml`.
   Verify: see the exact error (e.g. repo resolves, but `repo sync` 404s the
   fetch URL).
2. Confirm the real URL: the fetch is
   https://github.com/sunlightlinux/<name>. Check whether the repo is named
   `slkernel` (it is) — so `name` is correct.
   Verify: `git ls-remote https://github.com/sunlightlinux/slkernel` works,
   and the project's default branch is `main`, not `master`.
3. Identify the actual cause (e.g. the project's branch, not the manifest)
   before editing default.xml at all.
   Verify: a minimal `repo init ... && repo sync slkernel` succeeds.
```

Don't change `name`/`revision` on a hunch — confirm what `repo` actually
rejects first.

### Example 2 — Multi-Step Change with Verification

**User request:** "Set up a tagged release snapshot for `alpha-2.0`."

**What LLMs often do (all at once):**

Hand-edit `default.xml` to add `revision="<sha>"` to every project, change
`sync-j`, add a `<superproject>`, and commit — with no verification that the
SHAs are real or that `repo` accepts the result.

**What should happen (incremental):**

```
Plan:
1. Sync a clean tree at current main.
   Verify: `repo sync -j4` completes; `repo status` is clean.
2. Snapshot pinned SHAs into a separate file (don't touch default.xml):
   `repo manifest -r -o release-alpha-2.0.xml`.
   Verify: the file lists a real revision="<sha>" per project; `xmllint
   --noout release-alpha-2.0.xml` passes.
3. Sanity-check the snapshot resolves and reproduces the tree:
   `repo init -u <url> -m release-alpha-2.0.xml && repo manifest`.
   Verify: resolved revisions match the SHAs from step 2.
4. Commit `release-alpha-2.0.xml` (a new file), leaving default.xml tracking
   main.
   Verify: `git diff` shows only the new snapshot; default.xml unchanged.
```

The tracking manifest stays on `main`; the release lives in its own pinned
file.

---

## Anti-patterns Summary

| Principle | Anti-pattern | Fix |
|---|---|---|
| Think Before Coding | Silently picks `name`/`path`/`revision` for a new project | List the choices; ask which repo, path, and branch |
| Simplicity First | `<include>` splitting + group taxonomy for five projects | One `default.xml`; add `groups=` in place only when needed |
| Surgical Changes | Reorders, re-indents, adds `upstream=`/`clone-depth=` while adding one project | Add the single `<project>` line; nothing else moves |
| Goal-Driven | Changes `name` + `revision` on a hunch for a sync failure | Reproduce with `repo manifest`/`git ls-remote`, then fix the real cause |

## Key Insight

The "overcomplicated" examples aren't obviously wrong — `<include>` files,
groups, `upstream=`, and `clone-depth=` are all real, legitimate manifest
features. The problem is **timing**: they add structure before it's needed,
which:

- Makes the manifest harder to audit (it decides what everyone builds).
- Widens blast radius — a stray `sync-j`/`clone-depth` change affects every
  developer and CI run.
- Obscures the one line that actually answers the request.
- Risks supply-chain surprises (an extra remote or project is a new trust
  dependency).

The "simple" versions are easier to review, harder to get wrong, and can grow
later — once a second concrete requirement actually appears.

**Good manifest changes solve today's checkout while keeping `default.xml`
auditable and on `main`, not tomorrow's structure prematurely.**
