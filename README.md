# clice-llvm

Build infrastructure for the LLVM prebuilt packages that
[clice](https://github.com/clice-io/clice) depends on.

clice links against prebuilt LLVM/Clang static libraries. This repo stores
version-specific patches applied on top of upstream LLVM tags before building.
The actual build workflows live in the main `clice` repo (they depend on the
pixi environment for toolchain consistency).

## How builds work

1. A maintainer triggers `build-llvm.yml` in `clice-io/clice` with a target
   LLVM version (e.g., `21.1.8`).
2. The workflow clones upstream LLVM at `llvmorg-$VERSION`.
3. If `patches/$VERSION/` exists here, all `.patch` files are applied in order.
4. LLVM is built across a 14-job matrix (3 OS x Debug/RelWithDebInfo x native/cross).
5. `release-llvm.yml` prunes unused libraries and publishes to this repo's
   GitHub Releases as `$VERSION+rN`.

See the `/upgrade-llvm` skill in `clice-io/clice/.claude/commands/upgrade-llvm.md`
for the full step-by-step process.

## Patches

```
patches/
  21.1.8/
    0001-codegen-fix-illegal-std-template-specializations.patch
```

Each patch is a standard `git apply`-compatible diff against the upstream tag.
Numbered prefixes ensure deterministic application order.

### 21.1.8 patches

| # | Upstream | Description |
|---|----------|-------------|
| 0001 | [PR #160804](https://github.com/llvm/llvm-project/pull/160804) | Replace illegal `std::less`/`std::equal_to` specializations in RDFRegisters with custom types. Fixes libc++ 22 `static_assert(is_empty<Comparator>)` on macOS. |

## Release metadata

Each GitHub Release should document:

| Field | Example |
|-------|---------|
| LLVM source version | `llvmorg-21.1.8` |
| Release suffix | `+r2` |
| Compiler toolchain | clang 22.1.8 (conda-forge) |
| C++ stdlib (Linux) | libstdc++ 15.1.0 |
| C++ stdlib (macOS) | libc++ 22.1.8 (conda-forge) |
| C++ stdlib (Windows) | MSVC STL (toolset 14.42) |
| MSVC toolset | 14.42 (pinned via `ilammy/msvc-dev-cmd`) |
| Build script | `clice-io/clice@<commit>:scripts/build-llvm.py` |
| Patches applied | `patches/21.1.8/0001-*.patch` |
| CI run | `clice-io/clice/actions/runs/<id>` |

## Versioning

Prebuilt releases use `$VERSION+rN` (e.g., `21.1.8+r2`):
- `$VERSION` matches the upstream LLVM tag
- `+rN` increments when toolchain, patches, or build config change

## Future goals

- **Immutable releases**: pin published releases so they cannot be deleted
  (GitHub immutable release support). Every shipped clice version references
  a specific LLVM prebuilt that must remain downloadable.
- **Fully reproducible builds**: lock the exact pixi environment (pixi.lock
  hash) and CI runner image version in each release's metadata, so any
  release can be rebuilt bit-for-bit.
- **Automated upgrade tracking**: CI job that detects new LLVM point releases
  and opens a tracking issue with the API diff.
