# System Architecture

## 1. Overview

The Linux Corporate Infrastructure Lab is a virtualized
corporate infrastructure environment designed for practical
studies in Linux administration, networking, directory
services, monitoring and troubleshooting.

The environment will be hosted on a Proxmox VE hypervisor.

The infrastructure will use an isolated internal network,
protected by a virtual firewall.

---

## 2. Virtualization Platform

| Component | Configuration |
|---|---|
| Hypervisor | Proxmox VE |
| CPU | Intel Core i7-2600 |
| CPU Threads | 8 |
| RAM | 8 GB |
| Primary Storage | 240 GB SSD |
| Secondary Storage | 320 GB HDD |

Due to the limited hardware resources available in the
laboratory, virtual machines will be provisioned according
to their actual workload.

Non-essential virtual machines may remain powered off when
not required.

---

## 3. Network Architecture

The environment will use two logical networks.

### WAN

The WAN network provides connectivity between the laboratory
and the existing physical network.

```text
192.168.1.0/24