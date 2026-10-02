# Read-only inspection of the W154R PLUS

Only use these commands on equipment you own or administer. Replace placeholders with the address and interface of your own device. IPv6 link-local connections need an interface scope.

```sh
telnet 'fe80::YOUR_DEVICE_ADDRESS%eth0' 4444
```

**Firmware-specific security observation:** On the inspected unit TCP port **4444** presented an unauthenticated root `/bin/sh` prompt. Processes associated with **4446** (`telnetd -l /bin/sh`) and **9876** (`/etc/tzscript/login.sh`) were also observed. This does not establish default availability, external reachability, or behavior on other firmware versions. Restrict management-network access and treat exposed shell listeners as a security risk.

Once authorized shell access has been established, non-mutating inventory commands:

```sh
mdlcfg -e | grep -E '^export (SYS_HW_MODEL|SYS_HW_MODEL_TR069|SYS_HW_VENDOR|SYS_MANUFACTURER|SYS_CFG_VER|SYS_DEVICE_DESCRIPTION)='
uname -a
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/mtd
cat /proc/cmdline
cat /etc/version
mount
brctl show
ls /sys/class/net
netstat -lnt
```

Review output before publication. The full `mdlcfg -e` output, `/config` and `/data` may contain passwords, serial numbers, IMEIs, or local network data. This repository uses a small model-field allowlist and omits identifying values.

## Observed services

The snapshot included `mdlcfgd`, `tz_mgr`, `lan_mgr`, `wlan_mgr`, `hostapd`, `gpiod`, `exe_daemon` and `log_backup_daemon`. TCP listeners were present on 4444, 4446 and 9876. Their boot persistence and accessibility across network boundaries were not independently tested.
