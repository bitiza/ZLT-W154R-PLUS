# W154R PLUS web upload and local vendor updater — evidence as of 2026-10-03

The **direct local stock-updater path has now been successfully tested**, while the **web GUI upload path has not**. See the detailed [monitored vendor-package trial](idu-package-trial.md). This corrects the earlier documentation that called all vendor-package installation untested.

## Firmware entry points observed by inspection

- The saved Realtek legacy page `web/upload.htm` posts multipart `binary` to `/boafrm/formUpload`; a separate form refers to dual firmware.
- TOZED's frontend JavaScript accepts `.bin` / `.img`, submits multipart field `file`, and issues an upgrade request with the uploaded filename. The precise HTTP endpoint behavior has **not been exercised**.
- Local `libcmdlib.so` calls `packtoolpro` to extract a payload named `tzupdate` into `/tmp`, then invokes that updater with `--local --system --file` and LED options.

**Package-supplied updater caution:** This web backend may execute the updater carried inside a firmware package, not necessarily the stock on-flash executable. The direct test below used a verified copy of the saved stock updater.

## Signature and image validation boundaries

Inspection of the saved `libcmdlib.so` at function `util_verify_upload_file()` found a return-zero stub; messages referring to signature verification do not make that specific upload precheck a cryptographic authenticator. This does **not** establish universal absence of signatures in other binaries or model revisions.

The W154R PLUS package is a `packtoolpro` outer container with board/version metadata and per-payload MD5, carrying inner Realtek `cr6c` kernel and `r6cr` rootfs records. Both transport payloads and the bootloader's encoded-length SquashFS range use additive word checks. See [packtool](packtool.md) and [rootfs-checksum](rootfs-checksum.md). A syntactically readable outer container is not necessarily a flashable firmware image.

## Actual live acceptance and installation result

The owner's live trial used the verified board feature `ZLT W154R PLUS`, updater version `1.2.6`, framed factory kernel and clean v4 rootfs, and a stock updater record. The initial local command exited **114** on extraction. Inspection found just 14,916 KiB free under the persistent staging location `/data/web_upload`, too little for the 20,937,764-byte inner system record. After a temporary RAM-backed bind-mount was used to provide enough staging capacity, **the unchanged command returned zero**.

On bank 1, the writer changed the secondary kernel marker to `0x80000004` while primary partition hashes remained unchanged. After reboot, UART and Linux showed `bootbank=2`, root device `31:8`, completed vendor startup, and all 1,690 firmware regular-file hashes matching the clean v4 reference.

**Precisely what is now verified:** successful **direct invocation of the stock updater** on this particular IDU with sufficient staging resources, correct package framing and observed next-bank selection. **Not verified:** uploading the package through the HTTP GUI, interruption/recovery during a write, or compatibility with other model revisions.

## Risks and remaining work

The stock rootfs write branch was previously observed opening the selected MTD block device, skipping the 16-byte `r6cr` header and syncing the write. This vendor mechanism is not equivalent to `nc | dd` directly to a raw NAND character device, which can fail with short unaligned writes.

Keep the original MTD backups offline. Do not upload unknown packages through an endpoint capable of running embedded executables. For an independent audit, obtain an original W154R PLUS distribution package and compare the exact record set, transport fields and version rules. The X17U ODU differs: its inspected updater invokes signed SWUpdate and dm-verity, and it cannot use this Realtek image route; see [ODU comparison](odu-packtool.md).
