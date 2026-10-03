# ZLT X17U ODU: packtoolpro and signed OTA differences

**Scope:** Comparison against the W154R PLUS indoor unit. Based on the owner's saved AArch64 executables, read-only inspection of an authorized Unisoc UIS8520 X17U, and offline test execution. No successful ODU system modification, recovery install, or direct ODU firmware flash is established here.

## Shared outer package, separate firmware pipeline

The X17U `packtoolpro` binary produced a byte-identical outer format to the portable packtool implementation and the W154R PLUS's vendor tool for tested fixtures, including a file larger than two 512 KiB chunks. Its full extraction preserved input when tested with undocumented `-p`.

| Component | X17U SHA-256 |
| --- | --- |
| `packtoolpro` | `a5f836e3e2f25e9f0d77a7cf126d63a46e8a9444a5521293074f4dbc83415ce0` |
| `tzupdate` | `f0bbbdc94654c567533cc31dfcfe50c5f4f6f8965c66cbe54687c675551ce8bb` |

ODU programs are AArch64/musl; W154R PLUS IDU programs are MIPS/uClibc. A common tool name and outer container **do not mean that the firmware payloads are interchangeable**.

## Verified system storage and integrity

The inspected ODU runs Linux 5.15.185 on `uis8520-1h31-nand`, uses a large UBI region with one named `boot` volume and one named `system` volume, and mounts a SquashFS root via `/dev/dm-0`. Its kernel command line constructs dm-verity with SHA-256, 4 KiB blocks, and `restart_on_corruption`. Changing a protected filesystem block would require compatible verity metadata and trusted boot configuration; an arbitrary modified SquashFS is not comparable to the IDU's additive-checksum rebuild.

The observed UBI list includes a `recovery` volume, but this does not demonstrate a usable restoration procedure or a second complete Linux system slot. The owner's live U-Boot environment lacked an enabled `double_copy=y` setting. The backup bootloader and security partitions are not evidence of A/B system banks.

## Staged OTA updater — inspected, not installed

The inspected X17U `tzupdate` stage-2 path places an inner SWU as `/lcm/recovery/ota.swu` and checks it using SWUpdate and the vendor public key. The disassembly records:

```sh
swupdate -H uis8520:1.0 -e stable,normal -k /etc/swupdate_key/public.pem -c -i STAGED_FILE
```

On accepted check, it sets environment variable `mode=boot-recovery`, moving to a recovery installation workflow whose write and rollback behavior have **not been validated on-device**.

An offline QEMU test with the saved SWUpdate 2021.11.0 binary and an unsigned test SWU (no images or scripts) failed because `sw-description.sig` was absent. That supports a genuine SWUpdate signature-metadata requirement in this tested path; the necessary private signing key was not available. Separate permissive upload checks or outer packtool MD5 do **not** bypass this check.

Read-only board identification returned `ZLT X17U` from `devinfo 102025`. Another `sign` record check compared a value to a runtime `signKeyMd5` string; this equality check is distinct from SWUpdate's digital signature validation. Do not publish individual identifiers or secrets.

## Why bare --stage 2 rebooted

The supplied AArch64 disassembly of `tzupdate` shows:

- At `0x2d40`–`0x2d4c`, stage value 2 selects a distinct processing branch.
- At `0x2d5c`–`0x2d64`, the low three argument-flag bits must all be set (`--file`, `--local`, `--system`) to jump to the actual stage-2 processing at `0x31c0`.
- With only `tzupdate --stage 2`, it falls through to logging and `util_does_system_opt_reboot("TZUPDATE_REBOOT")` at `0x2d94`.

Therefore the observed bare-command restart is explained by **an unconditional reboot on that branch**, not by firmware bank selection. The supplied branch does not prove that every earlier parser instruction was side-effect-free.

## UBI sysdumpdb diagnostic note

Two inspected X17Us had the same software identification and 29-volume layout but different `ubi0_25` (`sysdumpdb`) update markers: one `0`, another `1`. The latter also showed one UBI bad physical eraseblock, though the bad block is not localized to `sysdumpdb`. For a dynamic UBI volume `corrupted=0` **does not prove that contents are intact**. The startup script `update-init.sh` calls `erase_partition "sysdumpdb"` within `wipe_data()` during factory-reset handling; an interrupted update is a plausible, **unproven** explanation for the marker.

**Do not run another `--stage` or recovery command merely to test bank switching.** The W154R PLUS's validated `tzupdate --switch` is a different implementation.

See [IDU stock updater result](idu-package-trial.md) and [outer packtool structure](packtool.md).
