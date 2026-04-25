# Rivet Compiler Reference Documentation

Plain-text references we need to own the whole compilation pipeline from Rivet
source to an executable Mach-O binary running on Apple Silicon (ARM64). No
LLVM, no assembler, no linker — we emit bytes directly.

Everything in this directory is checked in on purpose. When we hand-write an
ARM64 instruction encoder, we want to be able to look things up without
depending on a network.

## What's here

### ARM A64 instruction set

The base ISA. Everything we will ever emit is described by one of these.

- **`armv8a-isa.txt`** — *Armv8-A Instruction Set Architecture* (Arm Ltd, ARM062-948681440-3280, Issue 1.1, 2020). A readable overview of the instruction groups (data processing, load/store, branch, system) with encoding layouts. Best starting point for humans.
  - Source: <https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/Learn%20the%20Architecture/Armv8-A%20Instruction%20Set%20Architecture.pdf>
- **`arm-isa-overview.txt`** — *ARMv8 Instruction Set Overview* (Arm Ltd, PRD03-GENC-010197 v15.0, 2011). Older but more exhaustive: 112 pages of tables. The definitive per-mnemonic encoding reference short of the full Arm ARM (DDI 0487).
  - Source: <https://www.cs.princeton.edu/courses/archive/fall19/cos217/reading/ArmInstructionSetOverview.pdf>
- **`arm64-cheat-sheet.txt`** — Swarthmore CS31 one-page reference. Quick lookup of common mnemonics; don't trust it for encoding details.
  - Source: <https://www.cs.swarthmore.edu/~kwebb/cs31/resources/ARM64_Cheat_Sheet.pdf>

For the fully authoritative spec (thousands of pages, heavy tables) the full
*Arm Architecture Reference Manual for A-profile* (DDI 0487) lives at
<https://developer.arm.com/documentation/ddi0487/latest>. We haven't vendored
it — download and convert on demand if we need per-instruction encoding bits
that the above don't cover.

### Apple platform specifics

- **`apple-arm64.md`** — *Writing ARM64 code for Apple platforms* (Apple Inc., 2026). **This is the single most important ABI document for us.** It lists every way Apple deviates from the standard AAPCS64: reserved `x18`, frame-pointer rules, `long double` = `double`, 128-byte red zone, stack-argument alignment, variadic argument layout, C++ ABI differences, DIT for constant-time crypto.
  - Source: <https://developer.apple.com/documentation/xcode/writing-arm64-code-for-apple-platforms>
  - Fetched via the `.md` alternate: `https://docs.developer.apple.com/tutorials/data/documentation/xcode/writing-arm64-code-for-apple-platforms.md`

The generic AAPCS64 (everything Apple doesn't override) is ARM IHI 0055 at
<https://github.com/ARM-software/abi-aa/blob/main/aapcs64/aapcs64.rst>. Vendor
it when we start touching the calling convention in earnest.

### Mach-O executable format

Straight from the source. We emit Mach-O binaries, so we need byte-accurate
layouts of every header and load command.

- **`mach-o-loader.h`** — `<mach-o/loader.h>` from apple-oss-distributions/xnu. Defines `mach_header_64`, every `LC_*` load command, `segment_command_64`, `section_64`, flags (`MH_PIE`, `MH_NOUNDEFS`, `MH_DYLDLINK`, …), and every constant we need.
  - Source: <https://github.com/apple-oss-distributions/xnu/blob/main/EXTERNAL_HEADERS/mach-o/loader.h>
- **`mach-o-nlist.h`** — Symbol table entry format (`nlist_64`, `N_*` flags). We need this the moment we emit a relocatable object file.
  - Source: <https://github.com/apple-oss-distributions/xnu/blob/main/EXTERNAL_HEADERS/mach-o/nlist.h>
- **`mach-o-reloc.h`** — Relocation entry format (`relocation_info`, `scattered_relocation_info`, `ARM64_RELOC_*` types).
  - Source: <https://github.com/apple-oss-distributions/xnu/blob/main/EXTERNAL_HEADERS/mach-o/reloc.h>

### macOS system call interface

- **`xnu-syscalls.master`** — The syscall table source file. Column 1 is the
  syscall number. On ARM64 macOS, load the number (often OR'd with
  `0x2000000` in older docs, but that flag is for the BSD class — for the
  "unix" class the raw number works) into `x16`, arguments into `x0..x5`,
  then issue `svc #0x80`. On error the carry flag is set and `x0` holds the
  errno value.
  - Source: <https://github.com/apple/darwin-xnu/blob/main/bsd/kern/syscalls.master>
  - **Caveat:** Apple officially considers Darwin syscall numbers private. They
    don't change often in practice, but the supported API is libSystem.
    For a self-hosted compiler the pragmatic approach is: call
    `_exit`, `_write`, etc. in libSystem via the standard ABI, and reserve
    raw `svc` for a small startup stub.

### DWARF debugging information

- **`dwarf5.txt`** — *DWARF Debugging Information Format Version 5* (DWARF
  Standards Committee, 2017). ~19k lines of normative spec. We care about
  this because VALUES.md puts debuggability third on our priority list and
  GOALS.md says "Good DWARF support is absolutely essential."
  - Source: <https://dwarfstd.org/doc/DWARF5.pdf>
  - Relevant sections when we get there: §3 compilation units, §6.1 line
    number table, §6.2 line number program, §7 data representation (for
    producing the bytes), §4 type entries.

## What we still need (not vendored yet)

- **AAPCS64** (ARM IHI 0055) — the base calling convention Apple extends.
  Only needed once we implement calls from scratch.
- **Arm ARM (DDI 0487)** — per-instruction bit-level encoding reference.
  Only needed if our selected mnemonics aren't covered by `armv8a-isa.txt`
  or `arm-isa-overview.txt`.
- **Apple code-signing requirements** — ad-hoc signing (`LC_CODE_SIGNATURE`
  with CMS `SHA256` hashes) is required for binaries to run on Apple
  Silicon. We'll need this before `hello world` actually runs.

## Conversion notes

PDF sources were converted with `poppler-utils` (`pdftotext -layout`) via
`nix-shell -p poppler-utils`. The `-layout` flag preserves column structure,
which matters for encoding tables. Re-run with the same flag if re-fetching.
