# W154R PLUS — hardware and software inventory

All values in this document were obtained by read-only inspection of **one ZLT W154R PLUS on 2026-10-02**. They are not X17U vendor specifications.

## Platform

| Property | Observed value | Evidence |
| --- | --- | --- |
| System type | `RTL8197F` | `/proc/cpuinfo` |
| CPU | MIPS 24Kc V8.5, one logical processor | `/proc/cpuinfo` |
| PCI device | `10ec:b832` | `lspci -nn` |
| Kernel | `4.4.176`, MIPS; built 2026-02-28 | `uname -a`, `/proc/version` |
| SDK | Realtek SDK `v3.4.14-r` | `/etc/version` |
| BusyBox | `1.30.1` | `busybox` |
| Reported memory | 123,692 KiB | `/proc/meminfo` |
| Swap | None observed | `/proc/meminfo` |

The CPU's BogoMIPS value does **not** establish actual operating frequency. Board-level radio chipset identity, antenna count, channel widths, link-rate ceilings and flash-chip capacity were not established.

Observed kernel command line:

```text
console=ttyS0,38400 root=/dev/mtdblock3 root2=/dev/mtdblock8
```

## MTD layout

Sizes below are from `/proc/mtd` and sum to **115 MiB of named partitions**; this is not proof of total raw flash capacity. The primary rootfs was mounted read-only as SquashFS. Backup partitions exist, but failover and firmware-update behavior were not tested.

| Partition | Size | Name | Observed mount |
| --- | ---: | --- | --- |
| `mtd0` | 2 MiB | `boot` | — |
| `mtd1` | 3 MiB | `setting` | `/hw_setting` (YAFFS2) |
| `mtd2` | 4 MiB | `linux` | — |
| `mtd3` | 24.25 MiB | `rootfs` | `/` (SquashFS) |
| `mtd4` | 0.75 MiB | `jffs2 file` | `/jffs2` (JFFS2) |
| `mtd5` | 2 MiB | `boot2` | — |
| `mtd6` | 3 MiB | `setting2` | — |
| `mtd7` | 4 MiB | `linux2` | — |
| `mtd8` | 25 MiB | `rootfs2` | — |
| `mtd9` | 5 MiB | `config` | `/config` (YAFFS2) |
| `mtd10` | 5 MiB | `config_backup` | `/config_backup` (YAFFS2) |
| `mtd11` | 37 MiB | `data` | `/data` (YAFFS2) |

`/config` contains per-device settings including credentials. Neither it nor `/data` should be published wholesale.

## Network and Wi-Fi

At the time of inspection, `br0` contained `eth0`–`eth4`, `wlan0` and `wlan1`. `eth1` carried upstream traffic toward the outdoor unit and `wlan1` carried client traffic in that installation. Interface names do not prove physical port mapping.

Loaded modules included `rtl8192cd` and `rtk_wifi6`. `hostapd` ran for `wlan0` and `wlan1`. `iwlist` showed `wlan0` at 5 GHz channel 36 and `wlan1` at 2.4 GHz channel 1. These are **runtime observations**, not a complete regulatory-domain or product radio specification. Additional virtual `wlan1-vap*` interfaces existed but were not bridge members at inspection.

`br0` had IPv4 and IPv6 link-local addresses in `169.254.0.0/16` and `fe80::/10`. Exact addresses were withheld.

**Distinct X17U facts:** A separate X17U product listing advertises Unisoc V620, SA/NSA 5G, 2.5 Gbps networking and PoE; none should be inferred for the W154R PLUS. See [topology](topology.md) and [sources](sources.md).
