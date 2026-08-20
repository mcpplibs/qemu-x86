# qemu-x86

Cross-host builds of `qemu-system-x86_64` for the mcpp/xlings package index.

**Status: a question being asked, not a package.** Nothing here is published,
and nothing here should be added to the index until the workflow is green on
all five hosts.

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
| An index descriptor | Admission is a separate decision, made after five green legs |
| Committed binaries | The workflow builds what it checks, so nothing here is a blob whose provenance has to be trusted |
| A mirror upload | Mirroring an artifact that has not been shown to run would publish the untested thing faster |

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
