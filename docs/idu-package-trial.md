# Successful stock-updater vendor-package trial — W154R PLUS

**Date:** 2026-10-03. This is a report of the owner's monitored trial and subsequent console, flash-hash, and filesystem verification records. It documents a successful test on **one W154R PLUS**, not a universal upgrade method. The package and UART captures are private and not distributed.

## Tested package

| Field | Verified/reported value |
| --- | --- |
| Board feature | `ZLT W154R PLUS` from read-only `devinfo 102025` |
| Record version | `1.2.6` from read-only `devinfo 102003` |
| Records | `system` (framed factory kernel + corrected v4 rootfs); `tzupdate` updater |
| Package length | 21,014,508 bytes |
| Package SHA-256 | `19e35e737d24cbf7dad41554d698e2094a3c1b692bcfbd6b33fd72cf98cbff31` |
| Inner system payload SHA-256 | `af336618eee9f2ef76226c24c17744e2625fd335b967365eab9904f44d839a51` |
| v4 rootfs SHA-256 | `798c3ea2536f690438fcbd9a212abbcfb7a830fc7c5e5e83eaccb28c3a244b47` |

The rootfs transport header was `r6cr` with start 0, burn `0x00900000`, length 18,935,808. The rootfs's own bootloader checksum was preserved. For framing and checksums, see [packtool details](packtool.md).

## What actually happened

The device was initially running bank 1. All four rootfs/kernel MTD partition hashes were recorded before the test, and the transferred package and live updater were verified against the inspected files.

The stock command was:

```sh
/usr/bin/tzupdate --local --system --file /tmp/w154r-v4-vendor-trial.bin
```

**Attempt 1:** The updater returned **114** at its extraction-error path. The live `/data/web_upload` staging directory resided on YAFFS2, with only **14,916 KiB** free; the inner system image required **20,937,764 bytes**. All kernel/rootfs partition hashes remained unchanged after this failed attempt. No direct out-of-space message was captured, so insufficient staging capacity is a well-supported explanation, not a confirmed errno.

**Attempt 2:** After ensuring `/data/web_upload` was empty, the operator temporarily bind-mounted a RAM-backed directory over that path, verified sufficient RAM and package hashes, and reran the **same unchanged stock updater command**. It returned **0**. This was controlled staging for the test, not a documented general procedure for firmware updates. The temporary mount was removed before reboot.

## Complete partition comparison before reboot

| MTD | Resulting SHA-256 | Observation |
| --- | --- | --- |
| `mtd2` / primary kernel | `7f27571ffc26e8b2a0f60668751b1c31b6dfe086cfa82c0c59b4cd44e95520c6` | Unchanged |
| `mtd3` / primary rootfs | `4f4bf993dcea5339500904f11b7cc34d54bd210b445567d6339aaeb8e118221e` | Unchanged |
| `mtd7` / secondary kernel | `6057345b23fac9e88244fe8fe0d8cfc5151c5411864333ddfa0189a97c15a5b7` | Updated bank marker `0x80000004` |
| `mtd8` / secondary rootfs | `83bc02fd5cc55de56f77152099a78bac0450746528d61ac2c4a1008aed5e9db1` | Exact expected v4 image plus erased tail |

The bank 2 rootfs already contained v4 **before** the test; its unchanged hash shows that the final contents were correct, **not** that every byte was freshly programmed by the updater. The secondary kernel marker changed from `0x80000002` to `0x80000004`; that establishes the writer changed bank preference even without an explicit `--switch` or `--reboot` argument. Bank 1 was still running before restart.

## After reboot: successful bank 2 boot

UART showed `bootbank is 2, bankmark 80000004, forced:0`, and the kernel mounted SquashFS root from **device `31:8`**. Vendor initialization reached `Startup Ok`, `/proc/bootbank` reported `2`, and the added static BusyBox ran on-device.

A complete on-device audit read **all 1,690 regular firmware files**: all SHA-256 digests matched the clean extracted v4 tree. No missing or mismatched entries or SquashFS read errors were reported; the subsequent log check found no SquashFS/I/O errors.

**Outcome: the stock W154R PLUS updater accepted this package and the target bank booted successfully.** This confirms the tested direct updater route on this device, not the browser upload interface.

## Limits and practical safeguards

- The original W154R PLUS web-upload path remains **untested**. Do not claim that a browser upload has been demonstrated.
- The first failure demonstrates that `/data/web_upload` staging space matters; avoid deleting unrelated data merely to make room.
- Preserve factory MTD backups, isolate active vs inactive banks, check the full post-write hashes and retain UART logs.
- The reported test only exercised a package containing an already boot-tested v4 rootfs and the original factory kernel. Other image layouts, failures mid-write, or other board variants remain unverified.
- The ODU uses signed SWUpdate and dm-verity and **must not** receive this Realtek package.

See [container format](packtool.md), [rootfs integrity](rootfs-checksum.md), and [ODU update differences](odu-packtool.md).
