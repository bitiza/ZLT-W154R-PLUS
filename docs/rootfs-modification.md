# W154R PLUS rootfs modification: validated result and limitations

**Status as of 2026-10-03:** A modified SquashFS containing an extra static MIPSEL BusyBox 1.36.1 binary **successfully booted on bank 2**. The later v3 integrity audit found damaged vendor file contents even though the device booted; the supplied latest records report a corrected, clean v4 rebuild on both banks. Read [boot success and supporting output](boot-success.md) before attempting similar work.


> **Update, 2026-10-03:** A subsequent audit found corruption in the previously bootable v3 checksum-patched image, including altered bytes in a vendor executable and invalid XZ padding. The user's latest local records report a **clean rebuild (v4) installed on both rootfs banks**, with functioning access after reboot. This is a reported current state; the public repository does not contain private firmware images. See [checksum and v4 integrity findings](rootfs-checksum.md). Credentials and device identifiers are intentionally omitted.

The vendor package path has now succeeded with stock `tzupdate`, adequate staging space, and an already validated v4 image; consult [the monitored trial](idu-package-trial.md) before citing updater limitations. A successful direct local upgrade does not prove that the web GUI upload works.

## Observed baseline

- Realtek RTL8197F/MIPS 24Kc; Linux 4.4.176.
- Primary `mtd3` and secondary `mtd8` are SquashFS rootfs partitions.
- The original rootfs uses SquashFS 4.0, XZ, with 128 KiB blocks. The original superblock declares 18,307,631 filesystem bytes; subsequent non-erased bytes exist in the saved partition and their purpose remains unconfirmed.
- The rebuilt filesystem length recorded in the local notes is **18,931,712 bytes**; it adds `/usr/bin/busybox-full` while retaining the vendor BusyBox in `/bin`.
- Root mount evidence in the successful boot: `31:8 / / ro,relatime - squashfs /dev/root ro`. The added binary's help output identifies BusyBox 1.36.1.

## Integrity mechanism: do not confuse with dm-verity

Read-only inspection found no *active* standard dm-verity root path, device-mapper mount or usual verity parameters. This does **not** eliminate vendor checks. A separate agent's bootloader analysis describes a Realtek-specific rootfs checksum that interprets the standard SquashFS `mkfs_time` bytes at offset `0x08` as a transformed checksum length and expects a zero modulo-`65536` word sum. The corrected six bytes and successful bank 2 boot are documented in [boot success](boot-success.md).

The updater's local `util_verify_upload_file()` stub returning success, described in [web upgrade](web-upgrade.md), is **not equivalent** to bypassing bootloader checks. Image containers, inner headers, bank markers and NAND write integrity are separate issues.

## NAND and rollback precautions

The bank 2 partitions were entirely erased before the initial experiments. Direct Linux writes to `mtd7`/`mtd8` followed by a bank switch led to failed boot attempts; a corrected image later booted. The supplied earlier script did not record a complete `mtd8` readback hash. Raw partition captures lack NAND OOB/ECC data and are **not demonstrated restoration images**.

In the specific recorded recovery, removing bank 2's leading SquashFS signature allowed bootloader fallback to primary bank 1. That operation **erased NAND and is not a general-purpose or risk-free reset procedure**. Do not assume automatic fallback on every failure, or that holding the factory reset button selects a bank.

Future modifications should begin as offline, reproducible builds and preserve all existing files, modes, device nodes, links, architecture and compression settings. Verify exact output and planned recovery *before* a flash operation; avoid writing the currently mounted rootfs.

## Relevant documents

- [Boot success and checksum hypothesis](boot-success.md)
- [Flash geometry and recovery](flash-recovery.md)
- [Original backup constraints](backup.md)
- [Inspection](inspection.md)
