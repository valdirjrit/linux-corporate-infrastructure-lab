# Linux Corporate Infrastructure Lab

Laboratório de infraestrutura corporativa virtualizada desenvolvido para estudos práticos de **administração Linux, redes, firewall, serviços de diretório, DNS, compartilhamento de arquivos, monitoramento e troubleshooting**.

O ambiente será hospedado em um servidor **Proxmox VE**, utilizando máquinas virtuais para simular uma infraestrutura corporativa em um ambiente controlado e isolado.

---

## 📌 Status do Projeto

**Status:** Em desenvolvimento

| Componente               | Status                 |
| ------------------------ | ---------------------- |
| Proxmox VE               | 🟢 Base do laboratório |
| Rede WAN                 | 🟢 Concluído / Validado |
| Rede LAN                 | 🟢 Concluído / Validado |
| FW01 - OPNsense          | 🟢 Concluído / Validado |
| DC01 - Samba AD          | 🟡 Em Implementação     |
| SRV01 - File Server      | ⚪ Planejado            |
| MON01 - Zabbix / Grafana | ⚪ Planejado            |
| WKS01 - Windows          | ⚪ Planejado            |
| WKS02 - Linux            | 🟢 Concluído / Validado |

> O status dos componentes será atualizado conforme a implementação do laboratório avançar.

---

# 1. Visão Geral

O **Linux Corporate Infrastructure Lab** é um ambiente virtualizado de infraestrutura corporativa desenvolvido para estudos práticos de:

* Administração de sistemas Linux
* Administração de redes
* Firewall e roteamento
* Active Directory
* Samba
* DNS
* Kerberos
* LDAP
* Compartilhamento de arquivos
* Monitoramento de infraestrutura
* Troubleshooting
* Integração entre Windows e Linux
* Documentação de infraestrutura

A infraestrutura será hospedada sobre o **Proxmox VE** e utilizará uma rede interna isolada da rede física.

A comunicação entre a rede física e a rede interna será controlada por um **firewall virtual**.

---

# 2. Virtualization Platform

O laboratório será executado sobre um servidor **Proxmox VE**.

| Componente        | Configuração       |
| ----------------- | ------------------ |
| Hypervisor        | Proxmox VE 9.1.1   |
| CPU               | Intel Core i7-2600 |
| CPU Threads       | 4 cores / 8 Threads |
| RAM               | 8 GB               |
| Primary Storage   | 240 GB SSD         |
| Secondary Storage | 320 GB HDD         |

Devido aos recursos limitados disponíveis no laboratório, as máquinas virtuais serão provisionadas de acordo com a necessidade de cada serviço.

Máquinas virtuais que não forem necessárias durante determinada etapa poderão permanecer desligadas para reduzir o consumo de recursos.

---

# 3. Arquitetura da Rede

A infraestrutura utilizará duas redes lógicas:

* **WAN:** `192.168.1.0/24`
* **LAN:** `10.10.10.0/24`

## 3.1 WAN

A rede WAN fornecerá conectividade entre o laboratório e a rede física existente.

```text
Network: 192.168.1.0/24
Interface: vmbr0
```

O host Proxmox estará conectado à rede física através da bridge `vmbr0`.

---

## 3.2 LAN

A rede LAN será utilizada pelos serviços e estações do laboratório.

```text
Network: 10.10.10.0/24
Interface: vmbr1
Gateway: 10.10.10.1
```

A rede interna será isolada da rede física.

O firewall virtual será responsável pelo roteamento e controle do tráfego entre WAN e LAN.

---

# 4. Diagrama da Arquitetura da Rede

```text
                      REDE FÍSICA
                     192.168.1.0/24
                            |
                          vmbr0
                            |
                    +-------+-------+
                    |     FW01     |
                    |   OPNsense   |
                    |   Firewall   |
                    +-------+-------+
                            |
                          vmbr1
                            |
                     10.10.10.0/24
                            |
          +-----------------+-----------------+
          |                 |                 |
        DC01              SRV01             MON01
     10.10.10.10       10.10.10.20       10.10.10.30
          |                 |                 |
      Samba AD           File Server       Zabbix
        DNS                SMB             Grafana
          |
       +--+--+
       |     |
     WKS01 WKS02
    Windows Linux
      DHCP  DHCP
```

---

# 5. Interfaces de Rede

| Interface | Propósito                   | Rede             |
| --------- | --------------------------- | ---------------- |
| `vmbr0`   | WAN / Rede Física           | `192.168.1.0/24` |
| `vmbr1`   | Rede Interna do Laboratório | `10.10.10.0/24`  |

O Firewall virtual `FW01` terá interfaces conectadas em ambas bridges:

```text
FW01
├── WAN → vmbr0
└── LAN → vmbr1
```

---

# 6. Máquinas Virtuais

## 6.1 FW01 — Firewall / Router

| Parâmetro  | Configuração            |
| ---------- | ----------------------- |
| Hostname   | `FW01`                  |
| Platform   | OPNsense                |
| Role       | Firewall / Router       |
| WAN        | DHCP / Rede Física |
| LAN        | `10.10.10.1`            |
| Interfaces | `vmbr0` + `vmbr1`       |

### Responsabilidades

`FW01` será responsável por:

* Roteamento entre WAN e LAN
* Firewall
* Controle de tráfego
* Gateway da rede interna
* Isolamento da rede do laboratório
* Regras de acesso entre redes

---

## 6.2 DC01 — Domain Controller (Controlador de Domínio)

| Parâmetro | Configuração                |
| --------- | ---------------------------- |
| Hostname  | `DC01`                       |
| OS        | Debian Linux                 |
| Role      | Samba Active Directory / DNS |
| IP        | `10.10.10.10`                |
| Interface | `vmbr1`                      |
| GUI       | Nenhuma                      |

### Serviços

`DC01` fornecerá:

* Samba Active Directory
* DNS
* Kerberos
* LDAP
* Domain Authentication
* Users and Groups

O servidor será utilizado como controlador de domínio do ambiente de laboratório.

---

## 6.3 SRV01 — File Server (Servidor de Arquivos)

| Parâmetro | Configuração |
| --------- | ------------- |
| Hostname  | `SRV01`       |
| OS        | Debian Linux  |
| Role      | File Server   |
| IP        | `10.10.10.20` |
| Interface | `vmbr1`       |
| GUI       | Nenhuma       |

### Serviços

`SRV01` fornecerá serviços de compartilhamento de arquivos utilizando **SMB**.

O servidor também será utilizado para validar:

* Permissões de arquivos
* Permissões de compartilhamento
* Acesso por usuários do domínio
* Integração com o Active Directory

---

## 6.4 MON01 — Monitoring  (Servidor de Monitoramento)

| Parâmetro | Configuração  |
| --------- | ------------- |
| Hostname  | `MON01`       |
| OS        | Debian Linux  |
| Role      | Monitoring    |
| IP        | `10.10.10.30` |
| Interface | `vmbr1`       |
| GUI       | Nenhuma       |

### Serviços

`MON01` hospedará:

* Zabbix
* Grafana

O servidor será utilizado para monitoramento da infraestrutura e visualização de métricas.

---

## 6.5 WKS01 — Windows Workstation (Estação de Trabalho Windows)

| Parâmetro  | Configuração         |
| ---------- | --------------------- |
| Hostname   | `WKS01`               |
| OS         | Windows               |
| Role       | Corporate Workstation |
| Addressing | DHCP                  |
| Interface  | `vmbr1`               |
| GUI        | Sim                   |

`WKS01` será utilizado para validar a integração de uma estação Windows com o domínio Samba Active Directory.

Testes previstos:

* Ingresso no domínio
* Autenticação de usuários
* Resolução DNS
* Acesso a recursos compartilhados
* Aplicação de permissões

---

## 6.6 WKS02 — Linux Workstation (Estação de Trabalho Linux)

| Parâmetro  | Configuração     |
| ---------- | ----------------- |
| Hostname   | `WKS02`           |
| OS         | Debian Linux      |
| Role       | Linux Workstation |
| Addressing | DHCP              |
| Interface  | `vmbr1`           |
| GUI        | Sim               |

`WKS02` será utilizado para validar a integração de uma estação Linux com o domínio.

Testes previstos:

* Integração com Active Directory
* Autenticação de usuários
* Resolução DNS
* Acesso a recursos do domínio
* Acesso ao File Server

---

# 7. Resource Allocation

O laboratório possui recursos computacionais limitados.

A alocação inicial das máquinas virtuais será:

| VM    | vCPU |           RAM | Storage |
| ----- | ---: | ------------: | ------: |
| FW01  |    1 | 512 MB – 1 GB |    8 GB |
| DC01  |    2 |          1 GB |   16 GB |
| SRV01 |    1 |          1 GB |   16 GB |
| MON01 |    2 |          2 GB |   32 GB |
| WKS01 |    2 |         2 GB+ |  32 GB+ |
| WKS02 |    2 |         1 GB+ |  16 GB+ |

> Os recursos poderão ser ajustados conforme a utilização real de CPU, memória e armazenamento.

> Como o host possui 8 GB de RAM, nem todas as máquinas virtuais precisarão permanecer ligadas simultaneamente.

---

# 8. Storage Strategy

O armazenamento será dividido entre SSD e HDD de acordo com o tipo de utilização.

## 8.1 Primary Storage — SSD

O SSD de **240 GB** será priorizado para serviços que apresentem maior benefício com armazenamento de maior desempenho.

Possíveis utilizações:

* Máquinas virtuais de infraestrutura
* Serviços de monitoramento
* Sistemas operacionais
* Serviços utilizados com maior frequência

---

## 8.2 Secondary Storage — HDD

O HDD de **320 GB** poderá ser utilizado para:

* ISO images
* Backups
* Máquinas virtuais utilizadas com menor frequência
* Dados do laboratório
* Ambientes de teste
* Arquivos de documentação

A utilização definitiva do armazenamento será documentada conforme o laboratório for implementado.

---

# 9. Server Operating System Standard

Todos os servidores Linux da infraestrutura utilizarão instalação mínima, sem interface gráfica.

A administração será realizada principalmente de forma remota através de **SSH**.

## Standard Administration Tools

As principais ferramentas utilizadas serão:

```text
ssh
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
```

Outras ferramentas poderão ser adicionadas conforme a necessidade de cada serviço.

---

# 10. Workstation Standard

As estações de trabalho poderão utilizar interfaces gráficas.

O laboratório terá dois tipos de workstation:

| Workstation | Operating System | Purpose                    |
| ----------- | ---------------- | -------------------------- |
| WKS01       | Windows          | Windows Domain Integration |
| WKS02       | Linux            | Linux Domain Integration   |

A utilização de sistemas Windows e Linux permitirá testar um ambiente corporativo heterogêneo.

---

# 11. Implementation Plan

A infraestrutura será implementada de forma incremental.

Cada componente deverá ser validado antes da implementação da próxima etapa.

---

## Phase 1 — Network Foundation

### Tasks

* [X] Validate Proxmox network configuration
* [X] Validate `vmbr0`
* [X] Validate `vmbr1`
* [X] Deploy `FW01`
* [X] Configure WAN
* [X] Configure LAN
* [ ] Configure LAN gateway
* [ ] Configure firewall rules
* [ ] Validate routing
* [ ] Validate Internet connectivity
* [ ] Validate LAN isolation

---

## Phase 2 — Directory Services

### Tasks

* [ ] Deploy `DC01`
* [ ] Configure static IP
* [ ] Install Samba
* [ ] Provision Active Directory
* [ ] Configure DNS
* [ ] Configure Kerberos
* [ ] Validate domain functionality
* [ ] Create organizational structure
* [ ] Create users
* [ ] Create groups
* [ ] Test authentication

---

## Phase 3 — File Services

### Tasks

* [ ] Deploy `SRV01`
* [ ] Configure static IP
* [ ] Install Samba/SMB
* [ ] Configure shared folders
* [ ] Configure permissions
* [ ] Integrate with Active Directory
* [ ] Test domain user access
* [ ] Test permissions
* [ ] Validate Windows access
* [ ] Validate Linux access

---

## Phase 4 — Monitoring

### Tasks

* [ ] Deploy `MON01`
* [ ] Configure static IP
* [ ] Install Zabbix
* [ ] Configure Zabbix Server
* [ ] Configure monitoring agents
* [ ] Add infrastructure hosts
* [ ] Configure monitoring items
* [ ] Configure triggers
* [ ] Install Grafana
* [ ] Integrate Grafana with Zabbix
* [ ] Create dashboards

---

## Phase 5 — Workstations

### WKS01 — Windows

* [ ] Deploy Windows workstation
* [ ] Configure DHCP
* [ ] Configure DNS
* [ ] Validate connectivity with `DC01`
* [ ] Join workstation to domain
* [ ] Test domain authentication
* [ ] Test access to shared folders

### WKS02 — Linux

* [X] Deploy Linux workstation
* [X] Configure DHCP
* [ ] Configure DNS
* [ ] Validate connectivity with `DC01`
* [ ] Configure domain integration
* [ ] Test domain authentication
* [ ] Test access to shared folders

---

## Phase 6 — Operations and Troubleshooting

The final stage will focus on operational procedures and troubleshooting.

### Tasks

* [ ] Create monitoring alerts
* [ ] Simulate infrastructure failures
* [ ] Troubleshoot service failures
* [ ] Troubleshoot network connectivity
* [ ] Troubleshoot DNS
* [ ] Troubleshoot authentication
* [ ] Troubleshoot file permissions
* [ ] Document incidents
* [ ] Document root causes
* [ ] Test service recovery
* [ ] Document recovery procedures

---

# 12. Troubleshooting Scenarios

The laboratory will also be used to simulate common infrastructure incidents.

Examples include:

```text
Network connectivity failure
        ↓
DNS failure
        ↓
Domain authentication failure
        ↓
File server unavailable
        ↓
Incorrect permissions
        ↓
Monitoring alert
        ↓
Service failure
        ↓
Resource exhaustion
```

Possible scenarios include:

* High CPU utilization
* High memory utilization
* Disk space exhaustion
* Network interface failure
* DNS resolution failure
* Firewall rule misconfiguration
* Service stopped
* Incorrect file permissions
* Domain authentication failure
* SMB connectivity failure
* Monitoring agent failure

Each scenario should be documented with:

1. Symptoms
2. Investigation
3. Commands used
4. Root cause
5. Corrective action
6. Validation
7. Preventive action

---

# 13. Documentation Standards

Configuration changes and troubleshooting procedures will be documented throughout the implementation.

Documentation should include:

* Date
* Component
* Change performed
* Reason for change
* Commands executed
* Configuration changes
* Validation performed
* Result
* Problems encountered
* Resolution

The objective is to maintain a reproducible record of the laboratory.

---

# 14. Design Principles

The laboratory follows the following principles:

### Network Isolation

Keep infrastructure services isolated from the physical network whenever practical.

### Static Infrastructure Addressing

Use static IP addresses for infrastructure servers.

### Dynamic Client Addressing

Use DHCP for client workstations.

### Minimal Server Installation

Keep Linux infrastructure servers without graphical interfaces whenever practical.

### Remote Administration

Prefer remote administration through SSH.

### Incremental Implementation

Validate each infrastructure component before moving to the next stage.

### Documentation

Document configuration changes, incidents, troubleshooting procedures and root causes.

### Controlled Failure Simulation

Simulate infrastructure failures in a controlled environment.

### Resource Optimization

Avoid unnecessary resource consumption due to the laboratory's hardware limitations.

---

# 15. Project Structure

The documentation and configuration files will be organized as the project evolves.

```text
linux-corporate-infrastructure-lab/
│
├── README.md
│
├── docs/
│   ├── network.md
│   ├── proxmox.md
│   ├── opnsense.md
│   ├── samba-ad.md
│   ├── file-server.md
│   ├── zabbix.md
│   ├── grafana.md
│   └── troubleshooting.md
│
├── diagrams/
│   └── network.txt
│
├── scripts/
│   ├── linux/
│   └── windows/
│
├── configs/
│
└── screenshots/
```

> Diretórios e arquivos serão adicionados conforme cada etapa do laboratório for implementada.

---

# 16. Planned Services

| Hostname | Service                    | IP Address    |
| -------- | -------------------------- | ------------- |
| `FW01`   | OPNsense Firewall / Router | `10.10.10.1`  |
| `DC01`   | Samba AD / DNS             | `10.10.10.10` |
| `SRV01`  | SMB File Server            | `10.10.10.20` |
| `MON01`  | Zabbix / Grafana           | `10.10.10.30` |
| `WKS01`  | Windows Workstation        | DHCP          |
| `WKS02`  | Linux Workstation          | DHCP          |

---

# 17. Expected Learning Outcomes

Ao final da implementação, o laboratório deverá permitir a prática dos seguintes conhecimentos:

* Proxmox VE
* Virtualização
* Linux Server Administration
* TCP/IP
* Subnetting
* VLAN and network segmentation concepts
* Firewall
* Routing
* DNS
* DHCP
* Samba
* Active Directory
* Kerberos
* LDAP
* SMB
* Windows Domain Integration
* Linux Domain Integration
* Zabbix
* Grafana
* System Monitoring
* Troubleshooting
* Incident Documentation
* Infrastructure Documentation

---

# 18. Future Improvements

Conforme os recursos disponíveis e a evolução do laboratório, novas funcionalidades poderão ser adicionadas.

Possíveis melhorias:

* [ ] Additional Linux servers
* [ ] Additional Windows workstation
* [ ] Centralized logging
* [ ] Backup server
* [ ] Ansible automation
* [ ] Git-based configuration management
* [ ] Network monitoring
* [ ] Advanced firewall rules
* [ ] VLAN segmentation
* [ ] VPN
* [ ] High availability experiments
* [ ] Infrastructure-as-Code experiments
* [ ] Automated deployment
* [ ] Disaster recovery testing

---

# 19. Conclusion

O **Linux Corporate Infrastructure Lab** tem como objetivo fornecer um ambiente controlado para desenvolvimento e documentação de conhecimentos relacionados à infraestrutura de TI.

A implementação será realizada de forma gradual, começando pela fundação de rede e avançando para serviços de diretório, compartilhamento de arquivos, monitoramento, integração de estações Windows/Linux e troubleshooting.

Todo o processo será documentado para permitir a reprodução do ambiente e registrar os problemas encontrados, soluções aplicadas e conhecimentos adquiridos durante a implementação.

---

## Author

**Linux Corporate Infrastructure Lab**

Projeto pessoal de estudos e prática em infraestrutura de TI.
