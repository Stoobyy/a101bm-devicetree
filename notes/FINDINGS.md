# Findings — BALMUDA Phone A101BM (SoftBank), toward Magisk/root

Research log for a single unit: BALMUDA Phone A101BM, firmware `1.250PO.0675.a`,
bootloader unlocked via [unmuda](https://github.com/E2r7hN07Fl47/unmuda) (see
Credits below). Goal: determine whether rooting via a patched `boot.img`
(Magisk-style) is currently achievable, and if not, pin down exactly what's
missing.

## Hardware identification

- SoC platform: `lito` (Qualcomm **SM7250**, Snapdragon 765G class)
- Internal project codename: **MIZUKI** (Kyocera's name for the board, not
  Balmuda's)
- Exact board revision for this unit: **MIZUKI04 rev10**
  (`qcom,board-id = <0x04100005 0>`)

  Identified by reading `ro.boot.dtbo_idx` (`7`) off the live, running phone
  and matching it against the `dtbo-y` packing order in Kyocera's own
  `devicetree/Makefile` (0-indexed list of 10 overlay variants spanning
  MIZUKI00/02/03/04 across several revisions — see `devicetree/kyocera/Makefile`
  in this repo). This is a hard match from live device state, not a guess.

- `hw-revision` (fastboot getvar) is **not** a board identifier — confirmed
  from ABL source (`abl-reference/FastbootCmds.c`):
  ```c
  AsciiSPrint(StrSocVersion, sizeof(StrSocVersion), "%x", BoardPlatformChipVersion());
  FastbootPublishVar("hw-revision", StrSocVersion);
  ```
  It's the SoC silicon revision (`0x20000` = chip v2.0), unrelated to the
  MIZUKI board-ID scheme. (First instinct was to match it to a board variant —
  that was wrong; logged here so nobody repeats the detour.)

## What was tested, and what's actually dead vs. just documented-as-dead

The upstream `unmuda` docs already flag most of the fastboot/DIAG limitations
in comments. We re-verified each one empirically against a live unit rather
than taking the comments on faith:

| Path | Status | Evidence |
|---|---|---|
| `fastboot fetch <partition>` | **Unsupported by this bootloader** | `fastboot fetch boot_b out.img` → `Unable to get max-fetch-size. Device does not support fetch command.` Checked both in ABL (`is-userspace:no`) and fastbootd (`is-userspace:yes`) — neither advertises `fetch` in `getvar all`. |
| `adb root` / direct `dd` on `boot_a` | **Blocked** | `ro.build.type=user`; `adb root` → `adbd cannot run as root in production builds`; `dd if=/dev/block/.../boot_a` as shell (uid=2000) → `Permission denied`. |
| DIAG backdoor (`fs_sys_call.py`) reading `boot_a` | **Blocked, confirmed live** | `fs_sys_call.py /dev/block/bootdevice/by-name/boot_a out.bin` → `open(...) rejected: errno=13 (SELinux/path?)`. Matches the SELinux restriction already noted in `fs_sys_call.py`'s own docstring (`chkcode_block_device` only), but we actually triggered the denial rather than trusting the comment. |
| `fastboot boot <custom image>` | **Not attempted** — no verified image exists, see below | — |

**Net result:** there is currently no way, on this exact unit, to read
`boot`/`vendor_boot`/`dtbo` off the device, and no officially published or
community-published factory/OTA image exists either (checked Balmuda's OSS
page and XDA — see Credits/Sources). `fastboot boot` is the only theoretically
open path, and it's gated on having a verified image to boot in the first
place, which doesn't exist yet.

## The actual blocker on building one from source

Balmuda's official GPL/OSS releases (`tech.balmuda.com/jp/support/oss/`)
include Kyocera's own kernel (`msm-4.19`, GKI-flavored, Qualcomm common
Android kernel tree) and device-tree contributions — reproduced in
`devicetree/kyocera/` in this repo, straight from Balmuda's own
`opensource_A101BM_*.tar.gz`.

That tree is **not sufficient to build a complete device tree**. The common
overlay (`sm7250-MIZUKI-common.dtsi`) includes:

```
#include "../../qcom/proprietary/devicetree-4.19/qcom/lito-pmic-overlay.dtsi"
```

That path does not exist anywhere in either of Balmuda's released tarballs —
confirmed by grepping full tarball listings of both archives. This is
Qualcomm's own proprietary PMIC/power-rail configuration layer for the `lito`
reference platform, licensed to device OEMs under NDA. Balmuda/Kyocera were
never going to publish it; GPL only obligates disclosure of their own
modifications and the Linux kernel itself, not Qualcomm's separately-licensed
BSP.

**Important correction, logged so it isn't repeated:** the first instinct was
that this file might be a generic, silicon-level reference overlay safe to
borrow from another `lito`-platform device's public kernel dump (e.g. a
OnePlus Nord GPL release). That's wrong. Pulling the actual file from
`LineageOS/android_kernel_oneplus_sm7250` shows it's OEM/board-specific —
e.g. a `key_vol_up` binding tied to a specific GPIO on OnePlus's own PCB,
specific thermal ADC channel assignments, specific charger wiring. The
filename is shared by Qualcomm SDK convention; the content is not portable
across boards. Substituting another OEM's version would mean telling
Balmuda's hardware to use OnePlus's assumptions about physical pin wiring —
that's a real hardware-risk mistake (wrong GPIO-as-output / voltage-rail
config), not just a "won't boot" one.

**Conclusion:** without Kyocera's real PMIC overlay — which is not publicly
available anywhere, and can't be safely substituted — the device tree cannot
be completed with confidence. This is where the from-source-build path stops,
not for lack of effort but for lack of a legitimately obtainable piece.

## Open paths, if anyone picks this up later

- Someone with direct access to another A101BM/X01A unit with a *working*
  official update path might be able to pull `boot_a`/`vendor_boot_a` via a
  method not covered here (e.g. if a future firmware re-enables `fastboot
  fetch`, or if EDL programmer access is ever obtained for this platform).
- If Balmuda or Kyocera ever publish a newer OSS drop matching
  `1.220PO`–`1.280PO` specifically (current public drop stops at `1.200PO`),
  re-check whether the PMIC overlay gap is still present.
- Reverse-engineering the proprietary PMIC overlay from a dumped `dtbo`
  partition is theoretically possible (it's a compiled `.dtbo`, not source,
  but DTB is a fully recoverable format) — this just needs the partition dump
  this research couldn't get, which loops back to the dead ends above.

## Credits

This research builds directly on
**[unmuda](https://github.com/E2r7hN07Fl47/unmuda)** by **E2r7hN07Fl47** and
**Mamkin_Xakep**, with testing contributions from **radio_mudrec** — the tool
that actually gets the bootloader unlocked in the first place via the
`KcFastbootCheck()` / `LOOTBFCK` chkcode backdoor and CVE-2024-31317. None of
this would have been possible without that work. If you're starting from
scratch, go there first.

While using `unmuda`'s `cve31317.py` on native Windows (not WSL), we also hit
and fixed a real bug: `Path.write_text()` without `newline="\n"` lets Python's
default universal-newline translation turn every `\n` in the exploit's
byte-exact payload into `\r\n` on Windows, corrupting a format the exploit
comments themselves describe as byte-exact ("oddbyte"). This caused
inconsistent behavior (including an observed `android_system_crash` reboot)
rather than a clean success/fail. Worth upstreaming as a PR to `unmuda` for
other Windows-native (non-WSL) users.
