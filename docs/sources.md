# Sources, provenance and verification

## Direct primary evidence: W154R PLUS

The [hardware inventory](hardware.md) records **read-only observations from one physical ZLT W154R PLUS on 2026-10-02**, including `/proc/cpuinfo`, `/proc/mtd`, `/proc/meminfo`, `/etc/version`, `mdlcfg`, process and network listings. See [inspection](inspection.md). No raw configuration, credentials or identifying device data are published. Individual-firmware observations must not be represented as universal model specifications.

## Public product information: X17U (different device)

- [ZLT WiFi: Tozed ZLT X17U 5G Outdoor CPE Router](https://www.zltwifi.com/product/tozed-zlt-x17u-5g-outdoor-cpe-router-5g-odu/) — **seller/product listing**; advertises Unisoc V620, SA/NSA, 2.5 Gbps interface, PoE, and NAT/DHCP/firewall/VPN features for the **X17U ODU**, not W154R PLUS. Accessed 2026-10-02. This is not independently verified factory engineering documentation.

## Independent model distinction and deployment context

- [Madagascar ARTEC equipment approval list (PDF)](https://www.artec.mg/wp-content/uploads/2026/02/0302.pdf) — previously identified reference listing the W154R PLUS as a Tozed Kangwei Wi-Fi 6 wireless router and X17U separately as a 5G wireless router. This document has not been re-audited in this update; it does not establish board-level components.
- [Mevhare/airtel-odu-app](https://github.com/Mevhare/airtel-odu-app) — third-party project describing an Airtel deployment involving X17U and W154R PLUS. Deployment context only, not a vendor specification.

## Evidence rules

1. **Observed:** Verified on the inspected indoor unit; cite command/probe and inspection date.
2. **Advertised:** Reproduced in substance from the linked X17U product listing; label as X17U and avoid extrapolation.
3. **Unverified:** Distinguish possible capabilities from confirmed behavior. Do not silently promote assumptions into specifications.

No W154R PLUS manufacturer manual specific to the observed hardware/firmware revision has been verified here. Outstanding questions are tracked in [topology](topology.md) and [hardware](hardware.md).
