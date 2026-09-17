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
```
The Proxmox host is connected to this network through
vmbr0.

LAN

The laboratory's internal network will be isolated from the
physical network.

10.10.10.0/24

The internal network will be connected through vmbr1.

The virtual firewall will provide routing and security
between the WAN and LAN networks.

## 4. Network Interfaces
                     PHYSICAL NETWORK
                     192.168.1.0/24
                            |
                         vmbr0
                            |
                    +-------+-------+
                    |     FW01     |
                    |   Firewall   |
                    +-------+-------+
                            |
                         vmbr1
                            |
                     10.10.10.0/24
                            |
            +---------------+---------------+
            |               |               |
          DC01            SRV01           MON01
       10.10.10.10     10.10.10.20     10.10.10.30

## 5. Virtual Machines
FW01
Parameter	Configuration
Hostname	FW01
Platform	OPNsense
Role	Firewall / Router
WAN	DHCP / Physical Network
LAN	10.10.10.1
Interfaces	vmbr0 + vmbr1

FW01 will provide routing and firewall services between
the physical network and the internal laboratory network.

DC01
Parameter	Configuration
Hostname	DC01
OS	Debian Linux
Role	Samba Active Directory / DNS
IP	10.10.10.10
Interface	vmbr1
GUI	None

DC01 will provide:

Samba Active Directory
DNS
Kerberos
LDAP
Domain authentication
SRV01
Parameter	Configuration
Hostname	SRV01
OS	Debian Linux
Role	File Server
IP	10.10.10.20
Interface	vmbr1
GUI	None

SRV01 will provide file-sharing services using SMB.

MON01
Parameter	Configuration
Hostname	MON01
OS	Debian Linux
Role	Monitoring
IP	10.10.10.30
Interface	vmbr1
GUI	None

MON01 will host:

Zabbix
Grafana
WKS01
Parameter	Configuration
Hostname	WKS01
OS	Windows
Role	Corporate workstation
Addressing	DHCP
Interface	vmbr1
GUI	Yes

WKS01 will be used to validate Windows integration
with the Samba Active Directory domain.

WKS02
Parameter	Configuration
Hostname	WKS02
OS	Debian Linux
Role	Linux workstation
Addressing	DHCP
Interface	vmbr1
GUI	Yes

WKS02 will be used to validate Linux workstation
integration with the domain.

## 6. Resource Allocation

The laboratory has limited hardware resources.

Initial resource allocation:

VM	vCPU	RAM	Storage
FW01	1	512 MB - 1 GB	8 GB
DC01	2	1 GB	16 GB
SRV01	1	1 GB	16 GB
MON01	2	2 GB	32 GB
WKS01	2	2 GB+	32 GB+
WKS02	2	1 GB+	16 GB+

Resource allocation may be adjusted according to
actual workload and available system memory.

## 7. Storage Strategy

The 240 GB SSD will be prioritized for services that
benefit from faster storage.

The 320 GB HDD may be used for:

ISO images
Backups
Less frequently used virtual machines
Laboratory data
Test environments

Storage allocation will be documented as the environment
is implemented.

## 8. Server Operating System Standard

All Linux infrastructure servers will use a minimal
installation without a graphical interface.

Server administration will primarily be performed remotely
through SSH.

Standard administration tools include:

SSH
systemctl
journalctl
apt
ip
ss
dig
ping
df
du
top
ps

## 9. Workstation Standard

Workstations may use graphical interfaces.

The laboratory will contain both:

Windows workstation
Linux workstation

This allows testing of a heterogeneous corporate
environment.

## 10. Implementation Order

The infrastructure will be implemented in stages.

Phase 1 — Network foundation
 Validate Proxmox network
 Validate vmbr0
 Validate vmbr1
 Deploy FW01
 Configure WAN
 Configure LAN
 Validate routing
Phase 2 — Directory services
 Deploy DC01
 Configure static IP
 Install Samba
 Provision Active Directory
 Configure DNS
 Validate Kerberos
 Create users and groups
Phase 3 — File services
 Deploy SRV01
 Configure SMB
 Configure permissions
 Integrate with Active Directory
Phase 4 — Monitoring
 Deploy MON01
 Install Zabbix
 Configure monitoring agents
 Install Grafana
 Create dashboards
Phase 5 — Workstations
 Deploy WKS01
 Join Windows workstation to domain
 Deploy WKS02
 Join Linux workstation to domain
 Validate authentication
Phase 6 — Operations
 Create monitoring alerts
 Simulate infrastructure failures
 Perform troubleshooting
 Document incidents
 Test service recovery

## 11. Design Principles

The laboratory follows the following principles:

Keep infrastructure services isolated from the physical
network whenever practical.
Use static IP addresses for infrastructure servers.
Use DHCP for client workstations.
Keep Linux servers without graphical interfaces.
Prefer remote administration through SSH.
Validate each infrastructure component before moving to
the next stage.
Document configuration changes.
Simulate failures in a controlled environment.
Document troubleshooting procedures and root causes.
Avoid unnecessary resource consumption due to the
laboratory's hardware limitations.