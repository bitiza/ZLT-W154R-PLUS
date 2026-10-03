# Secondary bank flash attempts and observed recovery

**Dates:** 2026-10-02 through 2026-10-03.


> **Update, 2026-10-03:** A subsequent audit found corruption in the previously bootable v3 checksum-patched image, including altered bytes in a vendor executable and invalid XZ padding. The user's latest local records report a **clean rebuild (v4) installed on both rootfs banks**, with functioning access after reboot. This is a reported current state; the public repository does not contain private firmware images. See [checksum and v4 integrity findings](rootfs-checksum.md). Credentials and device identifiers are intentionally omitted.

## Source hierarchy

This chronology combines a supplied local agent's `IDU-FLASH-NOTES.md` with live owner-provided shell output reproduced in [boot success](boot-success.md). Reported actions are attributed as such; the successful mount and BusyBox execution are directly documented.

## Sequence

| Phase | Reported action or observation | Outcome |
| --- | --- | --- |
| Baseline | All `mtd5`–`mtd8` erased at initial backup | No second image at baseline |
| Attempt 1 | Kernel copied from `mtd2` to `mtd7`; rebuilt BusyBox rootfs written to `mtd8`; `tzupdate --switch` | No accessible bank 2 userspace; precise failure stage not captured |
| Recovery | Bootloader entered via UART; early block of secondary rootfs erased | Missing rootfs signature caused fallback to bank 1 in UART trace |
| Attempt 2 | Diagnostic rootfs with changes to early `rcS`; bank switch | Again no readable bank 2 initialization |
| Attempt 3 | Corrected header/checksum bytes applied to modified rootfs; reflashed bank 2 | **Successful:** bootbank 2; root mounted from device `31:8`; BusyBox 1.36.1 executes |

## What the original flash script did and did not verify

The prior transfer verified a source file hash, inspected the rootfs magic, and used erase and direct `dd`. Its post-flash hash command in the supplied notes hashed a source file rather than an independent complete partition readback. A partial first-64-byte kernel comparison does not prove complete kernel identity. No undocumented command should be interpreted as established safe firmware programming.

## Successful boot evidence

```text
cat /proc/bootbank
2

/proc/self/mountinfo root entry:
12 0 31:8 / / ro,relatime - squashfs /dev/root ro

dmesg:
VFS: Mounted root (squashfs filesystem) readonly on device 31:8.

BusyBox v1.36.1 (2026-10-02 18:18:03 EDT) multi-call binary.
```

The original kernel command line continues to list both `root=/dev/mtdblock3` and `root2=/dev/mtdblock8`; the VFS mount evidence identifies the actual active root. The most recent supplied uptime sample exceeds seven minutes on bank 2.

## Root-cause confidence

The notes associate the corrected image with the loader's rootfs checksum-length/checksum routine. This is a **well-supported, but not independently disassembly-audited explanation**. The available successful-boot logs establish compatibility of the final image; they do not establish whether the original failed images also had a separate NAND write problem.

The factory reset button did **not** produce recovery in the earlier owner report. The demonstrated recovery occurred via loader rejection of a deliberately invalidated secondary rootfs signature. Further writes have risk and should not be undertaken from this chronology alone.
