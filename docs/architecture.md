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

# 7. Alocação de Recursos

O laboratório possui recursos computacionais limitados.

A alocação inicial das máquinas virtuais será:

| VM    | vCPU |           RAM | Storage |
| ----- | ---: | ------------: | ------: |
| FW01  |    1 |          1 GB |   17 GB |
| DC01  |    2 |          1 GB |   17 GB |
| SRV01 |    1 |          1 GB |   17 GB |
| MON01 |    2 |          2 GB |   32 GB |
| WKS01 |    2 |         2 GB+ |  32 GB+ |
| WKS02 |    2 |         1 GB+ |  17 GB+ |

> Os recursos poderão ser ajustados conforme a utilização real de CPU, memória e armazenamento.

> Como o host possui 8 GB de RAM, nem todas as máquinas virtuais precisarão permanecer ligadas simultaneamente.

---

# 8. Estratégia de Armazenamento

O armazenamento será dividido entre SSD e HDD de acordo com o tipo de utilização.

## 8.1 Armazenamento Primário — SSD

O SSD de **240 GB** será priorizado para serviços que apresentem maior benefício com armazenamento de maior desempenho.

Possíveis utilizações:

* Máquinas virtuais de infraestrutura
* Serviços de monitoramento
* Sistemas operacionais
* Serviços utilizados com maior frequência

---

## 8.2 Armazenamento Secundário — HDD

O HDD de **320 GB** poderá ser utilizado para:

* ISO images
* Backups
* Máquinas virtuais utilizadas com menor frequência
* Dados do laboratório
* Ambientes de teste
* Arquivos de documentação

A utilização definitiva do armazenamento será documentada conforme o laboratório for implementado.

---

# 9. Sistema Operacional Server Padrão

Todos os servidores Linux da infraestrutura utilizarão instalação mínima, sem interface gráfica.

A administração será realizada principalmente de forma remota através de **SSH**.

## Ferramentas Padrão de Administração

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

# 10. Padrão das Estações de Trabalho

As estações de trabalho poderão utilizar interfaces gráficas.

O laboratório terá dois tipos de workstation:

| Workstation | Operating System | Purpose                    |
| ----------- | ---------------- | -------------------------- |
| WKS01       | Windows          | Windows Domain Integration |
| WKS02       | Linux            | Linux Domain Integration   |

A utilização de sistemas Windows e Linux permitirá testar um ambiente corporativo heterogêneo.

---

# 11. Plano de Implementação

A infraestrutura será implementada de forma incremental.

Cada componente deverá ser validado antes da implementação da próxima etapa.

---

## Fase 1 — Fundamentação de Rede

### Tarefas

* [X] Validate Proxmox network configuration
* [X] Validate `vmbr0`
* [X] Validate `vmbr1`
* [X] Deploy `FW01`
* [X] Configure WAN
* [X] Configure LAN
* [X] Configure LAN gateway
* [ ] Configure firewall rules
* [X] Validate routing
* [X] Validate Internet connectivity
* [X] Validate LAN isolation

---

## Fase 2 — Serviços de Diretório

### Tarefas

* [X] Deploy `DC01`
* [X] Configure static IP
* [ ] Install Samba
* [ ] Provision Active Directory
* [X] Configure DNS
* [ ] Configure Kerberos
* [ ] Validate domain functionality
* [ ] Create organizational structure
* [ ] Create users
* [ ] Create groups
* [ ] Test authentication

---

## Fase 3 — Serviços de Arquivos

### Tarefas

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

## Fase 4 — Monitoramento

### Tarefas

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

## Fase 5 — Estações de Trabalho

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

## Fase 6 — Operações e Soluções de Problemas

O Estágio Final terá foco em Procedimento operacionais e Solução de Problemas.

### Tarefas

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

# 12. Cenários de Solução de Problemas

O Laboratório será utilizado para simular erros e incidentes comuns em infrastrutura.

Examplos incluem:

```text
Falha de rede
        ↓
Falha de DNS
        ↓
Falha de autenticação de domínio
        ↓
Servidor de arquivos indisponível
        ↓
Permissões incorretas
        ↓
Alertas de monitoramento
        ↓
Falha de serviço
        ↓
Exaustação de recursos
```

Possíveis cenários incluem:

* Alta utilização de CPU
* Alta utilização de memória
* Falta de espaço em disco
* Falha de interface de rede
* Falha de resolução de DNS
* Falha de configuração de regra de firewall
* Serviço parado
* Permissão incorreta em arquivos
* Falha de autenticação no domínio
* Falha de conectividade de SMB
* Falha no agente de monitoramento

Cada cenário deve ser documentado com:

1. Sintomas
2. Investigação
3. Comandos utilizados
4. Causa raiz
5. Ação corretiva
6. Validação
7. Ação Preventiva

---

# 13. Padrões de Documentação

As alterações de configuração e os procedimentos de troubleshooting serão documentados ao longo da implementação.

A documentação deverá incluir:

* Data
* Componente
* Alteração realizada
* Motivo da alteração
* Comandos executados
* Alterações de configuração
* Validação realizada
* Resultado
* Problemas encontrados
* Resolução

O objetivo é manter um registro reproduzível do laboratório.

---

# 14. Princípios de Design

O laboratório segue os seguintes princípios:

### Isolamento de Rede

Manter os serviços de infraestrutura isolados da rede física sempre que possível.

### Endereçamento Estático da Infraestrutura

Utilizar endereços IP estáticos para os servidores de infraestrutura.

### Endereçamento Dinâmico dos Clientes

Utilizar DHCP para as estações de trabalho dos clientes.

### Instalação Mínima dos Servidores

Manter os servidores de infraestrutura Linux sem interfaces gráficas sempre que possível.

### Administração Remota

Priorizar a administração remota por meio do SSH.

### Implementação Incremental

Validar cada componente da infraestrutura antes de avançar para a próxima etapa.

### Documentação

Documentar alterações de configuração, incidentes, procedimentos de troubleshooting e causas raiz.

### Simulação Controlada de Falhas

Simular falhas de infraestrutura em um ambiente controlado.

### Otimização de Recursos

Evitar o consumo desnecessário de recursos devido às limitações de hardware do laboratório.

---

# 15. Estruturação do Projeto

A documentação e os arquivos de configuração serão organizados à medida que o projeto evoluir.

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

# 16. Serviços Planejados

| Hostname | Service                    | IP Address    |
| -------- | -------------------------- | ------------- |
| `FW01`   | OPNsense Firewall / Router | `10.10.10.1`  |
| `DC01`   | Samba AD / DNS             | `10.10.10.10` |
| `SRV01`  | SMB File Server            | `10.10.10.20` |
| `MON01`  | Zabbix / Grafana           | `10.10.10.30` |
| `WKS01`  | Windows Workstation        | DHCP          |
| `WKS02`  | Linux Workstation          | DHCP          |

---

# 17. Resultados Esperados de Aprendizag

Ao final da implementação, o laboratório deverá permitir a prática dos seguintes conhecimentos:

* Proxmox VE
* Virtualização
* Administração de Servidores Linux
* TCP/IP
* Subnetting (Sub-redes)
* Conceitos de VLAN e segmentação de rede
* Firewall
* Roteamento
* DNS
* DHCP
* Samba
* Active Directory
* Kerberos
* LDAP
* SMB
* Integração de estações Windows ao domínio
* Integração de estações Linux ao domínio
* Zabbix
* Grafana
* Monitoramento de Sistemas
* Troubleshooting
* Documentação de incidentes
* Documentação de infraestrutura

---

# 18. Melhorias Futuras

Conforme os recursos disponíveis e a evolução do laboratório, novas funcionalidades poderão ser adicionadas.

Possíveis melhorias:

* [ ] Servidor Linux adicionais
* [ ] Estação de trabalho Windows Adicional
* [ ] Centralização de Logs
* [ ] Servidor de backup
* [ ] Automação com ansible
* [ ] Gerenciamento de configurações baseadas em Git
* [ ] Monitoramento de rede
* [ ] Regras Avançadas de Firewall
* [ ] Segmentação de Redes com VLANs
* [ ] VPN
* [ ] Experimentos com alta disponibilidade (High availability)
* [ ] Experimento com Infraestrure-as-Code (IaC)
* [ ] Implantação automatizada
* [ ] Testes de recuperação de desastres (Disaster recovery)

---

# 19. Conclusão

O **Linux Corporate Infrastructure Lab** tem como objetivo fornecer um ambiente controlado para desenvolvimento e documentação de conhecimentos relacionados à infraestrutura de TI.

A implementação será realizada de forma gradual, começando pela fundação de rede e avançando para serviços de diretório, compartilhamento de arquivos, monitoramento, integração de estações Windows/Linux e troubleshooting.

Todo o processo será documentado para permitir a reprodução do ambiente e registrar os problemas encontrados, soluções aplicadas e conhecimentos adquiridos durante a implementação.

---

## Author

**Linux Corporate Infrastructure Lab**

Projeto pessoal de estudos e prática em infraestrutura de TI.
