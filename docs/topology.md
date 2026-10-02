# W154R PLUS and X17U: observed topology vs product claims

The W154R PLUS was inspected as the **indoor bridge and Wi-Fi access point** in a two-unit installation. The X17U is the **separate outdoor cellular unit**.

```text
Cellular 4G/5G network
        |
  ZLT X17U outdoor ODU
  (cellular uplink)
        |
  Ethernet / inter-unit link
        |
  ZLT W154R PLUS indoor router
  (observed bridge and Wi-Fi access point)
        |
  Wired and Wi-Fi clients
```

## X17U product listing, not W154R specification

The [ZLT WiFi X17U product page](https://www.zltwifi.com/product/tozed-zlt-x17u-5g-outdoor-cpe-router-5g-odu/) advertises:

| Advertised characteristic | Applies to | Verification status |
| --- | --- | --- |
| Unisoc V620 chipset | X17U outdoor ODU | Product-page claim |
| 5G NSA and SA support | X17U outdoor ODU | Product-page claim |
| 4G/5G communication | X17U outdoor ODU | Product-page claim |
| 2.5 Gbps network interface | X17U outdoor ODU | Product-page claim, link rate not measured here |
| PoE power supply via Ethernet | X17U outdoor ODU | Product-page claim |
| NAT, DHCP, firewall and VPN functions | X17U outdoor ODU | Listed capabilities; runtime roles depend on firmware/configuration |

The supplied URL is a **ZLT WiFi seller/product listing**, not a verified TOZED factory service manual. Treat its descriptions as advertised product information and corroborate against board/firmware observations when available.

## What we actually observed on the W154R PLUS

The indoor unit's `br0` bridged several Ethernet and Wi-Fi interfaces. In the tested configuration, the X17U was upstream and supplied the cellular connection and DHCP/DNS. That observed division of responsibility does **not** mean either component must always operate in the same mode. No claim is made that the indoor W154R PLUS contains a 5G modem or Unisoc V620.

## Not yet verified

- Actual physical Ethernet port mapping, negotiation rates and PoE wiring between units
- W154R PLUS radio chipset variants, antenna arrangement, rated Wi-Fi throughput and regional approvals
- Whether routing, DHCP, DNS, or VPN responsibilities change with firmware or operating mode
- X17U board revision and runtime feature support in the user's installation

See [hardware](hardware.md) for indoor hardware evidence and [sources](sources.md) for source attribution.
