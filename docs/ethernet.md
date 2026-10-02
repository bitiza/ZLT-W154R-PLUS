# Ethernet port mapping

Read-only evidence from the tested W154R PLUS firmware on 2026-10-02.

| Chassis role in firmware | Linux interface | Evidence | Live link at inspection |
| --- | --- | --- | --- |
| WAN / outdoor-unit uplink | `eth1` | `/etc/tzscript/tz_get_eth_lan_info.sh` and `get_lan_link_status.sh` | Up, 1 Gbit/s full duplex |
| LAN1 | `eth0` | Same vendor scripts | Down |
| LAN2 | `eth2` | Same vendor scripts | Up, 1 Gbit/s full duplex |
| LAN3 | `eth4` | Same vendor scripts | Down |

The scripts report ports in WAN, LAN1, LAN2, LAN3 order as `eth1,eth0,eth2,eth4`. `tzkw_cmd netport_status timeout,10` returned `port1,1000;port2,none;port3,1000;port4,none`, matching `/proc/eth*/link_status` for the listed interfaces. `/proc/rtl865x/port_status` showed switch ports 0 and 2 up at 1 Gbit/s and the other three switch ports down. The switch identified itself as `8367D` through `/proc/rtl_8367_chip_type`.

`eth3` exists and is in `br0`, but the vendor status scripts do not assign it to a named external port. It was down and had zero traffic during inspection. The script labels give a strong firmware mapping; the actual printed chassis labels and jack order have **not** been confirmed by moving a cable between ports.

`/sys/class/net/eth*/carrier` returned `1` for all five interfaces even when `/proc/eth*/link_status` reported links down, so the Realtek-specific proc status is the useful signal on this firmware. Packet counters alone show activity, not a reliable physical port label.

For a final physical check, record `/proc/eth*/link_status` and `tzkw_cmd netport_status timeout,10` before and after moving one LAN cable at a time. That would change the live connection and was outside this read-only session.
