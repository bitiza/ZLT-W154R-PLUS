# TOZED packtoolpro format — reverse-engineered reference

**Status (2026-10-03):** Offline behavior was tested with the saved W154R PLUS MIPS `packtoolpro` under QEMU and cross-checked against a portable Python implementation in the owner's private workspace. The subsequent [IDU vendor-package trial](idu-package-trial.md) succeeded on the test device. This page documents findings for this particular implementation, not all TOZED models.

## Binary identity and interface

Saved W154R PLUS `packtoolpro` SHA-256: `34c6877e641a6814ed34cd072543f578dc608836ff2ae33b38b3e37da25987e4`. The tool is little-endian MIPS32r2/uClibc. Its recorded CLI behavior:

| Syntax | Observed operation |
| --- | --- |
| `-m OUTPUT` | Make a container |
| `-f TEXT` | Container feature / board string; not validated as a numeric bitmap |
| `-a TYPE,FILE,VERSION` | Add a payload record; repeatable |
| `-l PACKAGE` | List metadata; **does not verify payload MD5** |
| `-r PACKAGE -t TYPE` | Extract selected type; verifies payload MD5 |
| `-r PACKAGE -n NAME` | Extract selected basename |
| `-e PACKAGE` | Extract all; **destructively truncates the package** |
| `-e PACKAGE -p` | Tested undocumented preserve flag; does not truncate the package |
| `-d TYPE,DIRECTORY` | Route extraction output by record type |

**Warning:** `-e` without `-p` destroyed the disposable input in testing; selection or extraction may overwrite local output filenames. Inspect only disposable copies in separate directories.

## Serialized container structure

The outer format has an encoded **24,160-byte index at the end**. Decoded index positions:

| Offset | Length | Meaning |
| --- | ---: | --- |
| 0 | 32 | ASCII decimal record count |
| 32 | 24,000 | Fifty 480-byte slots |
| 24,032 | 32 | Container version `1.1` |
| 24,064 | 32 | ASCII index length `24160` |
| 24,096 | 64 | Feature / board text |

Each record slot carries type (32 bytes), version (64), ASCII payload length (32), ASCII payload offset (32), lowercase MD5 hex text (64), and basename (256). Records are NUL-terminated and zero-padded.

### Byte encoding and 512 KiB chunk order

The tool substitutes printable ASCII bytes, leaving nonprintable bytes unchanged:

```python
def encode_byte(b):
    return 0x21 + ((b - 0x21 + 26) % 94) if 0x21 <= b <= 0x7e else b

def decode_byte(b):
    return 0x21 + ((b - 0x21 - 26) % 94) if 0x21 <= b <= 0x7e else b
```

Payloads are stored as **reversed logical 512 KiB chunks**, with bytes *within* each chunk kept in their original order before substitution. For original chunks `A | B | C` (where `C` is a short tail), physical order is `encode(C) | encode(B) | encode(A)`. A simple byte-rotation decode without reversing chunk order will not restore large firmware payloads. Larger payloads were observed to be written first; index record order need not match physical payload order.

This rotation and MD5 checking provide **obfuscation/integrity metadata, not cryptographic authentication**. Outer-container compatibility does not imply compatible inner images or signing rules.

## Tests from supplied research notes

- Portable Python and vendor tool produced byte-identical packages for three-record fixtures, reordered records and boundary lengths 524,287 / 524,288 / 524,289 / 1,048,593 bytes.
- Complete v4 SquashFS packaging round-tripped, with vendor-selected extraction returning the exact original bytes.
- A corrupted payload could still be listed by the vendor tool but failed selected extraction's MD5 check.
- The X17U ODU `packtoolpro` produced an identical *outer* package for cross-platform fixtures, though its executables are AArch64/musl rather than IDU MIPS/uClibc.

The Python tool in the owner's private research workspace was not included in this repository update. Do not treat a mention of the tool as a checked-in executable.

## W154R PLUS inner system record

An outer record of type `system` contains recognized Realtek transport images, rather than just a bare SquashFS:

```text
system payload:
  cr6c header (16 bytes) + declared factory kernel payload
  r6cr header (16 bytes) + corrected v4 SquashFS payload
```

Each Realtek header contains a four-byte magic and three big-endian 32-bit fields (start, burn, length). The kernel record's reported start is `0x80a00000`, burn `0x00500000`, and payload length **2,001,924**. For the successfully tested rootfs record, start `0`, burn `0x00900000`, and length **18,935,808** were accepted. The updater strips the `r6cr` header when installing the rootfs. Payload 16-bit big-endian additive checksums must sum to zero; the **bootloader separately validates the SquashFS-specific encoded length/checksum** described in [rootfs-checksum](rootfs-checksum.md).

During live query, decimal `devinfo 102025` supplied board text `ZLT W154R PLUS`, and `devinfo 102003` supplied firmware record version **`1.2.6`**. The separately displayed configuration revision `W154RPLUS-NG0002_1.0.05` is **not** equivalent to the updater's record-version value.

For the successful test package, the feature was exact board text, with `system` and a stock updater payload named `tzupdate`. Package metadata and inner checks must be evaluated for each model and firmware revision. A vendor update made for this IDU is **not** an X17U ODU update package.

See [tested installation](idu-package-trial.md), [vendor web interface status](web-upgrade.md), and [ODU comparison](odu-packtool.md).
