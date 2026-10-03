# Rootfs checksum, integrity failure and corrected v4 image

## Status — 2026-10-03

The owner-provided latest project notes report that **`rootfs-production-v4-safe.squashfs` was installed on both banks** of the ZLT W154R PLUS, with working authenticated management and UART access after reboot. This status comes from the supplied project logs, not an independent re-read of both NAND partitions in this GitHub update. Private credentials, individual device addresses, flash images and key-recovery notes are intentionally excluded.

| Property | Supplied result |
| --- | --- |
| v4 image size | **18,935,808 bytes** |
| Reported SHA-256 | `798c3ea2536f690438fcbd9a212abbcfb7a830fc7c5e5e83eaccb28c3a244b47` |
| SquashFS `bytes_used` | **18,931,784** |
| Rootfs partitions | `mtd3` (bank 1), `mtd8` (bank 2) |
| Added tool | Static MIPSEL BusyBox 1.36.1 |
| Access modification | Authenticated shell service configured at startup; no passwords published |

## Loader-specific rootfs integrity check

Offline disassembly of the saved Realtek stage-2 loader identifies a validation routine near virtual address **`0x8000ec0c`**. In that analysis the four bytes at SquashFS header offset 8 (normally `mkfs_time`) are interpreted **as a big-endian integer** to calculate a length:

```text
N = int.from_bytes(rootfs[8:12], "big") + 0x282
sum(big_endian_u16_words(rootfs[:N])) & 0xffff == 0
```

This is a **vendor bootloader convention**, not the standard SquashFS definition of `mkfs_time`. The original header's four bytes `01 17 5d 82` give `N = 0x01176004`, whereas the timestamp from the initial rebuilt filesystem (`60 af c0 6a`) gives an implausibly large reading range. The previously observed failed boots, the disassembly and the subsequent successful patched boot support this diagnosis.

The earlier image's zero 16-bit checksum alone was **not** a sufficient integrity test.

## Why the earlier working v3 image was unsafe

Later independent image inspection reported two failures in the earlier checksum-patched v3 family:

- A two-byte checksum adjustment previously treated as harmless padding actually modified bytes **inside the vendor executable `/usr/bin/tz_auto_relay`**.
- Another adjustment modified XZ padding associated with `/bin/AX_UDPs_97G`. The original audit recorded SquashFS extraction/read failures and live `I/O error` on that file.
- The rebuild also omitted **349 device nodes** found in the clean baseline.

Thus **a successful boot does not prove all contained binaries and compressed streams remain intact**. Treat v3 as superseded; do not distribute it as a validated firmware image.

## Corrected v4 design and validation

The supplied build report describes rebuilding from a clean tree with original files and metadata intact, then using only bytes **after SquashFS `bytes_used`**, within existing zero tail padding, for the checksum adjustment. Its checks:

1. Validate the little-endian SquashFS header and `bytes_used`.
2. Require zero padding and make output length NAND-page aligned.
3. Encode a checksum length covering the complete output image.
4. Place the two-byte adjustment at the end of the output image, outside filesystem data.
5. Verify the modulo-65536 word sum.
6. Fully re-extract and compare all file contents, symlinks, modes, ownership and device nodes before a flash.

The supplied audit states successful extraction of **1,690 files, 211 symlinks and 349 device nodes**, and a full comparison of 2,350 filesystem paths. Those are reported test results; this public repository does not carry the private images needed to rerun them.

## NAND streaming caveat

Raw NAND character writes require writes aligned to the 2 KiB NAND page size. Direct `nc | dd of=/dev/mtdN` pipelines can yield short reads and unaligned writes, producing `nand_do_write_ops: attempt to write non page aligned data`. Earlier aligned writes may have succeeded even if a later write was rejected. Prefer a verified local staging file and complete readback checks rather than streaming arbitrary socket chunks into NAND. Do not rewrite the active rootfs.

## Evidence limitations and disclosure

This document records a specific research unit and supplied diagnostics. It is **not** a tested generic recovery procedure. Flashing both banks removes a convenient intact-firmware fallback. Keep verified original partition backups offline and avoid publishing credentials, MAC/IMEI data, configuration, raw dumps, or private tooling. See [verified initial bank-2 boot](boot-success.md), [flash history](flash-attempt.md), and [flash geometry](flash-recovery.md).
