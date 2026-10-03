# Verified modified-rootfs boot — 2026-10-03

## Scope and evidence

An owner-operated W154R PLUS running the original Realtek Linux **4.4.176** kernel successfully booted **bank 2**, mounting a modified SquashFS filesystem that adds `/usr/bin/busybox-full`. This document records **observed terminal output** and separates it from the separate agent's bootloader reverse-engineering conclusions.

### Live verification

| Check | Observed output | Interpretation |
| --- | --- | --- |
| `cat /proc/bootbank` | `2` (repeated) | Runtime reports selected bank 2 |
| `cat /proc/self/mountinfo \| grep ' / '` | `12 0 31:8 / / ro,relatime - squashfs /dev/root ro` | Actual root device is **31:8**, i.e. **mtdblock8** |
| `dmesg` | `VFS: Mounted root (squashfs filesystem) readonly on device 31:8.` | Independent kernel confirmation |
| `/usr/bin/busybox-full --help` | `BusyBox v1.36.1 (2026-10-02 18:18:03 EDT)` | Added custom binary is present and executes |
| `uname -a` | `Linux rlx-linux 4.4.176 #2 Sat Feb 28 17:04:19 CST 2026 mips GNU/Linux` | Original kernel series runs with modified rootfs |
| `cat /proc/uptime` | `458.86 407.67` | Device remained running for at least ~459 seconds at this measurement |
| Direct Telnet | Interactive shell on TCP 4444 | Management access worked during verification |

The kernel command line remained `console=ttyS0,38400 root=/dev/mtdblock3 root2=/dev/mtdblock8`. **Both parameters appear even during a successful bank 2 boot**; the actual mount record `31:8` is the decisive evidence. The filesystem remained read-only, as expected for SquashFS.

### Network state at capture

`br0` contained `eth0` through `eth4` plus `wlan0` and `wlan1`. `/proc/eth0/link_status` returned `0`, which reports *that interface* down, not a loss of all network connectivity: the owner was connected through an interactive Telnet session. Presence of bridge members alone does not prove Wi-Fi association or end-to-end traffic.

## Attempt sequence (owner's accompanying notes)

1. **Attempt 1:** Original extracted rootfs rebuilt with an added static BusyBox; direct secondary kernel/rootfs programming; `tzupdate --switch` followed by an inaccessible/failed bank 2 startup.
2. **Attempt 2:** Additional diagnostic `rcS` changes; same failure reported.
3. **Attempt 3:** Four bytes of SquashFS header and two bytes of a proposed checksum adjustment changed; bank 2 mounted rootfs and executed added BusyBox successfully.

The local agent reported a resulting modified image of **18,931,712 bytes** and MD5 `f31521a8b02174bf4e00adc97526e807`. This hash is **from the supplied local notes**, not independently recalculated from the binary in this repository (no raw image is published). The purported six-byte difference is also based on those notes.

## Bootloader-specific checksum hypothesis

The owner's `IDU-FLASH-NOTES.md` attributes the initial failure to a loader routine at **0x8000ec0c** in the decompressed Realtek RTL8197F-VG v3.4.13 loader. The reported behavior is:

1. Check that the start of rootfs is `hsqs` / `sqsh`.
2. Read the 32-bit field at SquashFS offset `0x08`, byte-swap its value, and add `0x282` to obtain a **bootloader-specific checksum length**.
3. Sum big-endian 16-bit words over the resulting range; reportedly require modulo-`0x10000` sum of zero.

| Header bytes at `0x08` | Reported calculated range |
| --- | --- |
| Original: `01 17 5d 82` | `0x01176004` = 18,309,124 bytes |
| Initial rebuild: `60 af c0 6a` | `0x60afc2ec` ≈ 1.62 GB |

For the successful attempt the agent reports restoring bytes `01 17 5d 82` at `0x08` and adjusting bytes at `0x011660e0` from `00 00` to `5b 3c`. The reported 16-bit checksum of the first `0x01176004` bytes then equals zero.

**Distinction:** Offset `0x08` is the *standard SquashFS mkfs_time field*. The hypothesis is that this **particular Realtek loader reuses its bytes for a non-standard checksum-length computation**, not that SquashFS itself defines the field as a checksum length. The boot-success result strongly supports the reported fix, but a review of the precise disassembly and a controlled comparison are still needed to prove causality and rule out other differences. Garbled UART bytes alone do not prove an out-of-range NAND read.

## Recovery demonstrated before success

Earlier the owner-invalidated first erase region of the secondary rootfs at nominal offset `0x02b00000` with loader `NANDBE`. A subsequent loader trace showed scanning the empty secondary rootfs, reporting no signature, selecting bank 1 (`bank_mark=0x80000001`), and mounting primary `mtdblock3`. A later live check confirmed direct shell access with `/proc/bootbank=1`. **This is a demonstrated fallback under a missing-rootfs-signature condition, not proof of fallback under arbitrary corruption.** Reset-button recovery was reported ineffective and must not be documented as established.

## Outstanding verification

- Independently inspect the stage-2 loader disassembly at `0x8000ec0c` and audit checksum coverage, bounds, and error branches.
- Compare hashes of *read-back* NAND contents with the working image (the flash notes' earlier source-file checks did not prove complete NAND integrity).
- Confirm that bank 2 boot remains stable across planned reboots without changing settings.
- Investigate driver log warnings about MAC retrieval and regulatory database loading, comparing them with a stock boot.
- Keep device-specific identifiers, private NAND captures, modified binaries, and credential-containing configs outside public Git.
