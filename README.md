# kissat-builds

Per-platform prebuilt [KISSAT](https://github.com/arminbiere/kissat) SAT solver
binaries for [bmc4j](https://github.com/bmc4j/bmc4j)'s engine bundle.

KISSAT ships source only — no official prebuilt binaries — so this repo builds it
once per platform from a pinned upstream tag and publishes each binary as a GitHub
release asset, SHA-256-pinned. bmc4j's `bmc-engine-<platform>` jars fetch the
matching asset at their build time and verify it against the pinned checksum,
exactly like [bmc4j/jbmc-musl-builds](https://github.com/bmc4j/jbmc-musl-builds)
does for the musl jbmc binary.

## Pinned upstream

| | |
|---|---|
| Upstream | https://github.com/arminbiere/kissat |
| Version (tag) | `rel-4.0.4` |
| Commit | `8af8e56f174b778aef3aa45af9f739b2a5f492c2` |
| License | MIT (see [`LICENSE`](LICENSE)) |

The binaries are **unmodified** builds of the pinned source above (`./configure &&
make`, no source edits). The MIT `LICENSE` text is reproduced in this repo and is
uploaded alongside every binary asset (`KISSAT-LICENSE`) so it travels with the
binary.

## Platforms

One release per kissat version (tag `kissat-<version>-rN`); each release carries one
binary per platform plus a `SHA256SUMS` file and `KISSAT-LICENSE`.

| Platform asset | Runner | Build |
|---|---|---|
| `kissat-linux-x64` | `ubuntu-latest` | native `./configure && make` |
| `kissat-linux-arm64` | `ubuntu-24.04-arm` | native `./configure && make` |
| `kissat-linux-x64-musl` | `ubuntu-latest` + alpine container | static `./configure && make` in `alpine` |
| `kissat-macos-x64` | `macos-15-intel` | native `./configure && make` |
| `kissat-macos-arm64` | `macos-14` | native `./configure && make` |
| `kissat-windows-x64.exe` | `windows-latest` (MSYS2 / MinGW-w64) | `./configure && make` under MinGW |

Each built binary is smoke-checked with `kissat --version` (musl/arm in a container
if the runner can't execute it natively).

### windows-x64 — currently NOT shipped

KISSAT has no official Windows build. This repo attempts a MinGW-w64 build under
MSYS2; if it produces a working `kissat.exe` that runs `--version` it is shipped.

**As of `rel-4.0.4` the MinGW-w64 build does not succeed:** kissat's `src/resources.c`
unconditionally includes the POSIX header `sys/resource.h` (for `getrusage`-based
timing/memory accounting), which MinGW-w64 does not provide, so compilation fails with
`fatal error: sys/resource.h: No such file or directory`. Producing a Windows binary
would require **modifying** kissat's source (a Win32 shim for the resource layer), which
would violate this repo's unmodified-build guarantee. windows-x64 is therefore left as
the one open platform.

The other five platforms still ship — bmc4j treats kissat as optional per-platform and
builds its windows-x64 engine jar without it. If a future kissat release adds Windows
support (or a clean shim becomes available upstream), re-enabling it here is just a
green windows-x64 job; nothing downstream changes.

## Building / cutting a release

Dispatch the **Build kissat** workflow (`workflow_dispatch`), optionally overriding
the kissat tag and the release suffix. It builds every platform from the pinned tag,
smoke-checks each binary, computes `SHA256SUMS`, and publishes a single GitHub
release with all assets.
