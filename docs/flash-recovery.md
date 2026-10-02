# Flash layout and recovery evidence

Read-only inspection of one W154R PLUS running `W154RPLUS-NG0002_1.0.05`, plus offline analysis of its private backup. No flashing, reboot, bootloader entry, or recovery test was performed.

## NAND geometry

`/proc/nandinfo` reported a **128 MiB NAND chip**, 128 KiB erase blocks, 2 KiB pages, 128 bytes of out-of-band data per page, and zero bad blocks at inspection. The MTD sysfs entries reported type `nand`, the same erase/page/OOB sizes, and zero recorded ECC failures or corrected bits for all twelve partitions.

The partition table accounts for **115 MiB**, leaving **13 MiB outside the named MTD partitions**. Its purpose has not been established. The offsets below are *nominal cumulative offsets* derived from partition order and sizes; the actual NAND driver/bootloader physical mapping and bad-block handling have not been independently verified.

| MTD | Name | Nominal start | Nominal end (exclusive) | Size | Current contents / use |
| --- | --- | ---: | ---: | ---: | --- |
| 0 | `boot` | `0x0000000` | `0x0200000` | 2 MiB | Boot image; embedded compressed Realtek loader |
| 1 | `setting` | `0x0200000` | `0x0500000` | 3 MiB | Mounted `/hw_setting` (YAFFS2); hardware data |
| 2 | `linux` | `0x0500000` | `0x0900000` | 4 MiB | Kernel image; LZMA stream identified offline |
| 3 | `rootfs` | `0x0900000` | `0x2140000` | 24.25 MiB | Mounted `/`; SquashFS v4, xz |
| 4 | `jffs2 file` | `0x2140000` | `0x2200000` | 0.75 MiB | Mounted `/jffs2` |
| 5 | `boot2` | `0x2200000` | `0x2400000` | 2 MiB | Erased (`0xff`) |
| 6 | `setting2` | `0x2400000` | `0x2700000` | 3 MiB | Erased (`0xff`) |
| 7 | `linux2` | `0x2700000` | `0x2b00000` | 4 MiB | Erased (`0xff`) |
| 8 | `rootfs2` | `0x2b00000` | `0x4400000` | 25 MiB | Erased (`0xff`) |
| 9 | `config` | `0x4400000` | `0x4900000` | 5 MiB | Mounted `/config` (YAFFS2) |
| 10 | `config_backup` | `0x4900000` | `0x4e00000` | 5 MiB | Mounted `/config_backup` (YAFFS2) |
| 11 | `data` | `0x4e00000` | `0x7300000` | 37 MiB | Mounted `/data` (YAFFS2) |

The live `/proc/bootbank` value was `1`. The kernel command line selected `/dev/mtdblock3` as `root` and listed `/dev/mtdblock8` as `root2`; `/` was mounted from the primary rootfs. All bytes of `mtd5`–`mtd8` in the backup were `0xff`, so there was no usable second image at capture time. `config_backup` is a separate configuration partition and should not be mistaken for a firmware bank.

## Recovery routes indicated by evidence

| Route | Evidence on this unit | Status |
| --- | --- | --- |
| Running-system firmware update | The extracted rootfs contains `tzupdate` and `tz_upgrade_client`. The updater's strings show a local system-upgrade mode, `/proc/bootbank` handling, image checks, and signature-verification messages. | Present in firmware; accepted package format, signing requirements, and rollback behavior not tested. |
| Realtek bootloader network transfer | A gzip stream inside saved `mtd0` expands to loader code containing `TFTP`, `IPCONFIG`, `AUTOBURN`, `LOADADDR`, and NAND command strings. | Candidate bootloader recovery path; bootloader entry, network address, physical port, accepted image format and write behavior untested. |
| Realtek bootloader serial transfer | The same loader contains `XMOD`/XMODEM strings. The Linux command line and getty use `ttyS0,38400`. | Candidate path; UART header, voltage, pinout, bootloader baud rate, and transfer behavior untested. |
| External NAND programming | The backup contains all named MTD data areas. | Hardware recovery possibility only; no procedure validated. The backup excludes NAND OOB data and is not a proven directly flashable image. |

The loader also contains checksum and rootfs/signature error strings. Their presence does not establish which W154R PLUS firmware package will pass validation. Because the second bank is empty, automatic fallback should not be assumed.

Related RTL8197F hardware may have bootloader TFTP capabilities, but addresses and commands from another model must not be copied to this unit without independent verification. No model-specific recovery procedure has yet been established.
