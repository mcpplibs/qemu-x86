# qemu-x86

Cross-host builds of `qemu-system-x86_64` for the mcpp/xlings package index.

**Status: published.** `9.2.4-1` is released here, mirrored to
`xlings-res/qemu-x86` on GitCode, and admitted to the index as `xim:qemu-x86`
(openxlings/xim-pkgindex#665).

```
xlings install qemu-x86 -y
```

⭐ The reason it exists is `mcpplibs/openarch`: its `examples/switch` now boots
on `x86_64-none-elf` through `mcpp run` and prints `switch ok`, which is the one
claim a working `--version` does not make.

## Why this repository exists

The index has a stated bar for admitting an emulator, written down in
`qemu-riscv`'s descriptor: prebuilt binaries for the five host targets the index
serves — linux x64, linux arm64, darwin x64, darwin arm64, win32 x64 — from one
versioned release, each asset carrying a checksum sidecar.

`xim:qemu-arm` and `xim:qemu-riscv` clear that bar because xPack publishes them.
xPack builds QEMU per target family and has **no x86 build**; qemu.org ships a
Windows installer only, and macOS and Linux are served by distribution packages.
So no upstream clears it, and `xim:qemu-x86` cannot be a repackaging job the way
its two siblings are. It has to be built.

## Why the order is "build first, publish later"

⭐ **A failure here costs a red cross. A failure after publication costs a
package that installs and does not work.**

The tempting order is to write the descriptor, mirror the assets and open the
index pull request, and to discover only then that one host does not build.
GitHub's hosted runners happen to cover exactly the five hosts the index serves,
so all five legs can be attempted in one matrix before anything is published —
and the hardest of them fails early rather than last.

⚠️ **The hardest is Windows, and it is hardest for a structural reason.** QEMU on
Windows is built under MSYS2/MinGW rather than MSVC: its build system is meson
and its sources assume a POSIX-ish toolchain. What comes out is a native PE that
needs a set of MinGW runtime DLLs beside it — so the packaging question there is
a DLL directory rather than an rpath, and it is not answered by copying what the
Linux leg does.

## What is deliberately not here

| | |
|---|---|
| Committed binaries | The workflow builds what it checks, so nothing here is a blob whose provenance has to be trusted |
| Firmware pruning | Which `pc-bios` blob a machine type loads is a runtime question. Answering it by deleting until something breaks is how a payload ends up missing one blob on somebody else's machine — this was attempted once during development and reverted |

## What "it works" means here

A built binary is not evidence. Each leg that can do so boots a minimal
multiboot image that prints over the serial port and powers the machine off, and
asserts the printed line — the same discipline the rest of this ecosystem's
end-to-end tests use: assert the product, not the exit code.

⚠️ Two legs cannot do that and say so rather than passing quietly: an arm64 or
macOS runner has no host assembler that emits 32-bit x86 ELF, so on those the
emulator's own `--version` is the whole of the check. That is a weaker claim,
and it is recorded as one.

## The single-target build

```
--target-list=x86_64-softmmu
```

QEMU's full build is dozens of system emulators plus tools, documentation and UI
backends. The index needs one emulator, not a distribution, and a single target
with the UI and tools disabled is a small fraction of the work and of the
resulting size.

The version is pinned to the series `qemu-arm` and `qemu-riscv` already carry, so
that a user who installs all three gets one QEMU generation rather than two.

## The payload is self-contained, and that was measured rather than assumed

The first build produced binaries that ran on the machine that built them and
nowhere else: the Linux legs left `libpixman-1`, the glib family, `libz` and
`libzstd` to the host, and the darwin legs hardcoded `/opt/homebrew/opt/...`.

Every non-system shared library is now bundled beside the emulator and reached
through `$ORIGIN/../lib` (linux), `@loader_path/../lib` (darwin) or the
executable's own directory (win32, where PE has no runtime search path). The
workflow asserts it: what remains outside the payload must be core libc and
system frameworks, nothing else.

Measured on the released linux-x64 asset — fifteen objects resolve, twelve from
the payload's own `lib/`, and the two that cross the boundary are `libc.so.6`
and `libm.so.6`. That measurement is what lets `xim:qemu-x86` declare no `deps`.

⚠️ **The first attempt at that measurement measured nothing.** It was written as
`ldd <bin> 2>/dev/null | grep -v <payload>`, which printed nothing and read
exactly like "nothing escapes" — the `ldd` on the PATH was a shell script that
failed to parse, and it failed on stderr. The numbers above come from
`LD_TRACE_LOADED_OBJECTS=1` invoked on the loader directly.

⚠️ **Bundling also broke the macOS leg silently.** Its pipelines end in `grep`,
an empty-matching `grep` returns 1, `pipefail` promotes it and `errexit` turns
it into an exit — so the leg exited 1 after reporting `1564/1564`. `set +e`
around the bundling blocks fixes it.

## Archive layout

The archives are flat: `bin/` and `share/` at the top level, no wrap directory.

⚠️ That is a choice the consumer has to know about. xim extracts in place, into
a directory it also uses for other things, so a descriptor cannot move the
extraction directory wholesale the way `qemu-arm`'s can — `xim:qemu-x86` moves
the two entries by name, and asserts the emulator *before* the move rather than
after, because the source directory is shared.
