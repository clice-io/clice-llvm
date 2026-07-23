# clice-llvm

Build infrastructure for the LLVM prebuilt packages that
[clice](https://github.com/clice-io/clice) depends on.

## How it works

1. `build-llvm.yml` in `clice-io/clice` triggers a build with a target
   LLVM version (e.g., `21.1.8`).
2. The workflow clones upstream LLVM at `llvmorg-$VERSION`.
3. Patches from `patches/$VERSION/` in this repo are applied in order.
4. LLVM is built across a 14-job matrix (3 OS x configurations).
5. `release-llvm.yml` prunes unused libraries and publishes to this repo's
   Releases as `$VERSION+rN`.

See `/upgrade-llvm` in `clice-io/clice/.claude/commands/upgrade-llvm.md`.

## Patches

```
patches/
  21.1.8/
    0001-codegen-fix-illegal-std-template-specializations.patch
```

Patches are `git apply`-compatible diffs against the upstream tag.
Numbered prefixes ensure deterministic order.

### 21.1.8

| # | Upstream | Description |
|---|----------|-------------|
| 0001 | [PR #160804](https://github.com/llvm/llvm-project/pull/160804) | Fix illegal `std::less`/`std::equal_to` specializations in RDFRegisters. Required for libc++ 22 builds. |

## Versioning

Releases use `$VERSION+rN` (e.g., `21.1.8+r2`):
- `$VERSION` = upstream LLVM tag
- `+rN` increments when toolchain, patches, or build config change

## Reproducibility

When a release is published (e.g., `21.1.8+r2`), this repo is tagged at
the commit used for that build. To reproduce:

```bash
git clone --branch "21.1.8+r2" https://github.com/clice-io/clice-llvm.git
```

During development (tag doesn't exist yet), `build-llvm` uses `main`.

## Release metadata

Each Release should record:

| Field | Example |
|-------|---------|
| LLVM source | `llvmorg-21.1.8` |
| Compiler | clang 22.1.8 (conda-forge) |
| libstdc++ (Linux) | 15.1.0 |
| libc++ (macOS) | 22.1.8 |
| MSVC toolset (Windows) | 14.42 |
| Patches | `patches/21.1.8/0001-*.patch` |
| Build CI run | `clice-io/clice/actions/runs/<id>` |
| clice branch | `chore/llvm-prebuilt-r2@<commit>` |
