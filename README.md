# ZLT W154R PLUS — Device Research

Community-maintained technical documentation for the **TOZED ZLT W154R PLUS** indoor Wi-Fi router, as used alongside a **ZLT X17U** outdoor cellular unit in one inspected installation.

## Observed W154R PLUS inventory

| Property | Observation | Evidence |
| --- | --- | --- |
| Reported model | `ZLT W154R PLUS` |
| Configuration version | `W154RPLUS-NG0002_1.0.05` |
| Platform | Realtek RTL8197F; MIPS 24Kc V8.5, one logical processor | `/proc/cpuinfo` |
| Linux kernel | 4.4.176 (build dated 2026-02-28) | `uname`, `/proc/version` |
| SDK / BusyBox | Realtek SDK v3.4.14-r / BusyBox 1.30.1 | `/etc/version`, `busybox` |
| Linux-reported memory | 123,692 KiB | `/proc/meminfo` |
| NAND flash | 128 MiB reported; 115 MiB across named MTD partitions | `/proc/nandinfo`, `/proc/mtd` |
| Inspected role | Indoor Ethernet bridge / Wi-Fi access point | Network inspection |


## Documents

- [Hardware, storage, network interfaces](docs/hardware.md)
- [Read-only inspection and observed services](docs/inspection.md)
- [Read-only MTD backup findings and restore limitations](docs/backup.md)
- [Ethernet port mapping](docs/ethernet.md)
- [NAND geometry and evidence-based recovery candidates](docs/flash-recovery.md)
- [W154R PLUS ↔ X17U topology and responsibility](docs/topology.md)
- [Sources, evidence classification and open questions](docs/sources.md)

## New backup findings (2026-10-02)

All twelve MTD partitions were captured locally through read-only devices, totaling **115 MiB**; this does not include NAND OOB metadata and is not a validated restore image. **The nominal second firmware bank (`mtd5`–`mtd8`) was fully erased** on the inspected unit. Images and identifying data are intentionally excluded. See [backup notes](docs/backup.md) and [hardware inventory](docs/hardware.md).

## Ethernet and recovery research

Vendor scripts map WAN to `eth1`, LAN1 to `eth0`, LAN2 to `eth2`, and LAN3 to `eth4`. These are **firmware-reported roles**, not a chassis-jack mapping verified by moving cables. A live NAND inventory reports 128 MiB total with 115 MiB in named partitions; the additional 13 MiB has an unknown purpose. Extracted bootloader strings mention TFTP and XMODEM, but neither recovery route has been tested. See [Ethernet](docs/ethernet.md) and [flash recovery](docs/flash-recovery.md).

## X17U product-page reconciliation

The [ZLT WiFi product listing for the X17U](https://www.zltwifi.com/product/tozed-zlt-x17u-5g-outdoor-cpe-router-5g-odu/) describes **a separate outdoor 4G/5G ODU** using **Unisoc V620**, supporting **5G SA/NSA**, a **2.5 Gbps network interface** and **PoE power**. It additionally advertises NAT, DHCP, firewall and VPN functionality. These are **X17U listing claims**, not independently verified W154R PLUS hardware facts. The page is a product listing on ZLT WiFi; it is not presented here as an independent factory engineering datasheet. See [topology](docs/topology.md).

## Publication and safety

This public repository deliberately excludes the supplied `private/` notes, raw configuration, firmware images, serial numbers, IMEIs, MAC addresses, device-specific IPs, Wi-Fi credentials and passwords. Do not publish full `mdlcfg -e` output or files from `/config` without sanitizing them. The inspection commands are read-only; run them only on devices you administer. One firmware image may expose management services that other revisions do not.

Independent community project; not affiliated with or endorsed by TOZED or the website hosting the X17U listing.
