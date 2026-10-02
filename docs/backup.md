# Local backup notes

On 2026-10-02, all twelve W154R PLUS MTD partitions were read through their `/dev/mtdNro` devices and saved locally. The `ro` device nodes are the read-only MTD interfaces. The transfer used `gzip` and `xxd` over the existing telnet shell; decoding and hashing happened on the host. The device was not flashed or reconfigured.

The capture totaled **120,586,240 bytes (115 MiB)**. Each file's size matched `/proc/mtd`, and the host's SHA-256 hashes were checked against the manifest. A separate MD5 calculation on the device matched the saved `mtd0` boot image. The nominal secondary firmware bank (`mtd5`–`mtd8`) was fully erased on the inspected unit.

The backup contains credentials, calibration and unique identifiers. Keep it outside Git, restrict file access, and review it before copying elsewhere. It is a raw partition capture, **not a tested restore image**. In particular, the method does not capture NAND out-of-band metadata, and no restore or boot test was performed.
