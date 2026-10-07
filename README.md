# A101BM Device Tree Research

Research notes and recovered device-tree source for the **BALMUDA Phone A101BM** (SoftBank model).

Goal: figure out whether Magisk-style rooting via a custom `boot.img` is achievable on this phone. This repo documents what was checked, what was found, and exactly where the effort currently stalls - so the next person doesn't repeat the same dead ends.

This is not a tool, not a flashable image, and not a rooting guide.

## Start here if you haven't unlocked the bootloader yet

You want **[unmuda](https://github.com/E2r7hN07Fl47/unmuda)** by **E2r7hN07Fl47** and **Mamkin_Xakep**, not this repo.

`unmuda` is the actual bootloader-unlock tool. It uses CVE-2024-31317 plus a Kyocera service-mode backdoor (`KcFastbootCheck()` / the `LOOTBFCK` chkcode magic) to unlock the bootloader on this exact phone. Everything in this repo starts *after* that's done.

Full credit to them for the unlock itself - this repo only exists because their work made it possible to get far enough to ask the next question.

If you're on Windows without WSL, see [`notes/unmuda-windows-newline-fix.md`](notes/unmuda-windows-newline-fix.md) first - it covers a real bug we hit and fixed in `cve31317.py`.

## What's in this repo

- **`devicetree/kyocera/`** - Kyocera's own device-tree source for this phone, as published in Balmuda's official GPL/OSS release (`tech.balmuda.com/jp/support/oss/`, firmware `1.200PO.0605.a`, the newest version Balmuda has published source for). Reproduced here, under the same terms Balmuda released it, so nobody has to re-download and re-extract a 300MB archive to find these specific files.
- **`abl-reference/`** - Relevant excerpts from the bootloader (ABL/UEFI) source, also from Balmuda's OSS release. Explains how fastboot variables and device-tree selection actually work on this platform, without reading through 190MB of EDK2 source to find it.
- **`notes/FINDINGS.md`** - The actual research log. How the exact hardware revision was identified, what was tested against the live device (not just read from comments), and precisely which missing piece blocks building a complete, bootable device tree from public sources alone.

## The short version

**Bootloader unlock:** solved, by `unmuda`.

**Reading boot/vendor_boot off the device** to get a known-good image to patch: not currently possible.
- `fastboot fetch` isn't supported by this bootloader
- No root (production build, `adb root` refused)
- The DIAG service-mode backdoor is SELinux-restricted to the `chkcode` partition only

All three confirmed empirically, not assumed.

**Building a boot image from Balmuda's public kernel/DT source:** blocked.

The device tree depends on a Qualcomm proprietary PMIC overlay (`lito-pmic-overlay.dtsi`) that Balmuda never published - GPL doesn't require it, since it's Qualcomm's separately-licensed IP, not Balmuda's own code. It also can't be safely substituted from another device: despite the shared filename convention across every Snapdragon-765G (`lito`-platform) phone, the actual contents are board-specific wiring, not a generic reference file. Full reasoning for why substitution would risk real hardware misconfiguration (not just a failed boot) is in `FINDINGS.md`.

## If you want to pick this up

See the "Open paths" section at the end of `FINDINGS.md` - a newer Balmuda OSS drop, a dumped `dtbo` partition to reverse-engineer, or access to a second unit could all unblock this differently.

## License / provenance

- `devicetree/kyocera/` and `abl-reference/` are Kyocera/Balmuda's own published source, redistributed under the terms of their original release (GPL-covered kernel contributions; EDK2/Tianocore components under their own BSD-style license). Original headers are preserved in each file.
- `notes/` is this project's own research writing.
