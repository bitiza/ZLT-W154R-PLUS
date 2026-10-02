# ZLT W154R PLUS — Device Research

Community-maintained technical documentation for the **TOZED ZLT W154R PLUS** indoor Wi-Fi router, as used alongside a **ZLT X17U** outdoor cellular unit in one inspected installation.

> **Scope:** The W154R PLUS and X17U are different devices. Hardware and software values below come from read-only inspection of **one W154R PLUS on 2026-10-02**, unless separately attributed. They may vary with hardware revision, market and firmware.

## Observed W154R PLUS inventory

| Property | Observation | Evidence |
| --- | --- | --- |
| Reported model | `ZLT W154R PLUS` | `mdlcfg` |
| Configuration version | `W154RPLUS-NG0002_1.0.05` | `mdlcfg` |
| Platform | Realtek RTL8197F; MIPS 24Kc V8.5, one logical processor | `/proc/cpuinfo` |
| Linux kernel | 4.4.176 (build dated 2026-02-28) | `uname`, `/proc/version` |
| SDK / BusyBox | Realtek SDK v3.4.14-r / BusyBox 1.30.1 | `/etc/version`, `busybox` |
| Linux-reported memory | 123,692 KiB | `/proc/meminfo` |
| Storage | 115 MiB total across named MTD partitions (not a measured flash-chip capacity) | `/proc/mtd` |
| Inspected role | Indoor Ethernet bridge / Wi-Fi access point | Network inspection |

**Do not treat the X17U's 5G modem chipset, cellular bands, or 2.5 Gbps outdoor Ethernet specification as specifications for this indoor router.**

## Documents

- [Hardware, storage, network interfaces](docs/hardware.md)
- [Read-only inspection and observed services](docs/inspection.md)
- [W154R PLUS ↔ X17U topology and responsibility](docs/topology.md)
- [Sources, evidence classification and open questions](docs/sources.md)

## X17U product-page reconciliation

The [ZLT WiFi product listing for the X17U](https://www.zltwifi.com/product/tozed-zlt-x17u-5g-outdoor-cpe-router-5g-odu/) describes **a separate outdoor 4G/5G ODU** using **Unisoc V620**, supporting **5G SA/NSA**, a **2.5 Gbps network interface** and **PoE power**. It additionally advertises NAT, DHCP, firewall and VPN functionality. These are **X17U listing claims**, not independently verified W154R PLUS hardware facts. The page is a product listing on ZLT WiFi; it is not presented here as an independent factory engineering datasheet. See [topology](docs/topology.md).

## Publication and safety

This public repository deliberately excludes the supplied `private/` notes, raw configuration, firmware images, serial numbers, IMEIs, MAC addresses, device-specific IPs, Wi-Fi credentials and passwords. Do not publish full `mdlcfg -e` output or files from `/config` without sanitizing them. The inspection commands are read-only; run them only on devices you administer. One firmware image may expose management services that other revisions do not.

Independent community project; not affiliated with or endorsed by TOZED or the website hosting the X17U listing.
