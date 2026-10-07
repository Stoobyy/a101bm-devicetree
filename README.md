# A101BM Device Tree Research

Research notes and recovered device-tree source for the **BALMUDA Phone
A101BM** (SoftBank model), aimed at figuring out whether Magisk-style rooting
via a custom `boot.img` is achievable on this phone — and documenting exactly
where that effort currently stalls, so the next person doesn't have to redo
the same dead ends.

This is not a tool, not a flashable image, and not a rooting guide. It's a
record of what was checked, what was found, and what's still missing.

## Start here if you haven't unlocked the bootloader yet

**You want [unmuda](https://github.com/E2r7hN07Fl47/unmuda) by
[E2r7hN07Fl47](https://github.com/E2r7hN07Fl47) and Mamkin_Xakep, not this
repo.** `unmuda` is the actual bootloader-unlock tool — it uses
CVE-2024-31317 plus a Kyocera service-mode backdoor
(`KcFastbootCheck()` / the `LOOTBFCK` chkcode magic) to unlock the
bootloader on this exact phone. Everything here starts *after* that's done.
Full credit to them for the unlock itself; this repo exists only because
their work made it possible to get far enough to ask the next question.

If you're on Windows without WSL, see
[`notes/unmuda-windows-newline-fix.md`](notes/unmuda-windows-newline-fix.md)
for a real bug we hit and fixed in `cve31317.py` — worth knowing before you
run it.

## What's in this repo

- **`devicetree/kyocera/`** — Kyocera's own device-tree source for this
  phone, as published in Balmuda's official GPL/OSS release
  (`tech.balmuda.com/jp/support/oss/`, firmware `1.200PO.0605.a` — the newest
  version Balmuda has published source for). Reproduced here under the same
  terms Balmuda released it, for anyone who doesn't want to re-download and
  re-extract a 300MB archive to find these specific files.
- **`abl-reference/`** — Relevant excerpts from the bootloader (ABL/UEFI)
  source, also from Balmuda's OSS release, that explain how `fastboot`
  variables and device-tree selection actually work on this platform. Useful
  for anyone trying to understand `KcFastbootCheck()`'s neighborhood, or how
  `hw-revision`/`dtbo_idx` are derived, without reading 190MB of EDK2 source.
- **`notes/FINDINGS.md`** — The actual research log: how the exact hardware
  revision was identified, what was tested against the live device (not just
  read from comments), and precisely which missing piece blocks building a
  complete, bootable device tree from public sources alone.

## The short version

- Bootloader unlock: solved, by `unmuda`.
- Reading `boot`/`vendor_boot` off the device to get a known-good image to
  patch: **not currently possible** — `fastboot fetch` isn't supported by
  this bootloader, there's no root, and the DIAG service-mode backdoor is
  SELinux-restricted to the `chkcode` partition only. All three confirmed
  empirically, not assumed.
- Building a boot image from scratch using Balmuda's public kernel/DT source:
  **blocked** — the device tree depends on a Qualcomm proprietary PMIC
  overlay (`lito-pmic-overlay.dtsi`) that Balmuda never published (GPL
  doesn't require it — it's Qualcomm's separately-licensed IP) and that
  **cannot be safely substituted** from another device, because despite the
  shared filename it's board-specific wiring, not a generic reference file.
  Details and the reasoning for why substitution would actually risk real
  hardware misconfiguration (not just a failed boot) are in `FINDINGS.md`.

If someone picks this up later with a different angle — a newer Balmuda OSS
drop, a dumped `dtbo` partition to reverse-engineer, or access to a second
unit — `FINDINGS.md` has the open-paths section.

## License / provenance

- `devicetree/kyocera/` and `abl-reference/` are Kyocera/Balmuda's own
  published source, redistributed under the terms of their original release
  (GPL-covered kernel contributions; EDK2/Tianocore components under their
  own BSD-style license). Original headers are preserved in each file.
- `notes/` is this project's own research writing.
