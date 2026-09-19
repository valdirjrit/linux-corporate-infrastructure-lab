# Network Architecture

## 1. Overview

This document defines the network architecture used by the
Linux Corporate Infrastructure Lab.

The environment is designed to simulate a small corporate
network running on a virtualized infrastructure.

The network will provide connectivity between servers,
workstations and infrastructure services while maintaining
a simple and predictable addressing scheme.

---

## 2. Network Parameters

| Parameter | Configuration |
|---|---|
| Network | `10.10.10.0/24` |
| Network Mask | `255.255.255.0` |
| Default Gateway | `10.10.10.1` |
| DNS Server | `10.10.10.10` |
| Domain | `corp.lab` |
| DHCP Range | `10.10.10.100 - 10.10.10.200` |

---

## 3. IP Address Allocation

Static addresses will be used for infrastructure servers.

| Hostname | IP Address | Role |
|---|---:|---|
| FW01 | `10.10.10.1` | Firewall / Gateway |
| DC01 | `10.10.10.10` | Samba AD / DNS |
| SRV01 | `10.10.10.20` | File Server / SMB |
| MON01 | `10.10.10.30` | Zabbix / Grafana |

Workstations will initially obtain their addresses
through DHCP.

| Hostname | Addressing | Role |
|---|---|---|
| WKS01 | DHCP | Windows Workstation |
| WKS02 | DHCP | Linux Workstation |

---

## 4. Network Topology

```text
                         INTERNET
                            |
                            |
                     +------+------+
                     |    FW01     |
                     |  Firewall   |
                     | 10.10.10.1  |
                     +------+------+
                            |
                            |
                    10.10.10.0/24
                            |
          +-----------------+-----------------+
          |                 |                 |
     +----+----+       +----+----+       +----+----+
     |  DC01   |       |  SRV01   |       |  MON01  |
     |  Debian |       |  Debian  |       |  Debian |
     |10.10.10.10|     |10.10.10.20|     |10.10.10.30|
     +---------+       +---------+       +---------+
          |
          |
     +----+----------------------+
     |                           |
+----+----+                 +----+----+
|  WKS01  |                 |  WKS02  |
| Windows |                 | Debian  |
|  DHCP   |                 |  DHCP   |
+---------+                 +---------+
```
---
## 5. WAN Network

The WAN side of the firewall connects to the existing physical network.

- Network: `192.168.1.0/24`
- Gateway: `192.168.1.1`
- Firewall WAN: `192.168.1.3`
- Addressing: DHCP