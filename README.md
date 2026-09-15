# Linux Corporate Infrastructure Lab

> Laboratório de infraestrutura corporativa desenvolvido para estudos práticos, documentação técnica e demonstração de competências em administração de ambientes Linux, redes, serviços de infraestrutura, monitoramento e troubleshooting.

---

## 📌 Sobre o projeto

Este projeto consiste na construção de um ambiente corporativo virtualizado, utilizando principalmente servidores Linux e estações Windows/Linux.

O laboratório foi projetado para simular situações encontradas em ambientes reais de infraestrutura e suporte, permitindo praticar não apenas a implantação dos serviços, mas também:

- Administração de servidores Linux
- Redes e serviços de infraestrutura
- Active Directory
- DNS e DHCP
- Gerenciamento de usuários e grupos
- Compartilhamento de arquivos
- Monitoramento de infraestrutura
- Troubleshooting
- Análise de incidentes
- Recuperação de serviços
- Segurança e hardening
- Automação e scripts
- Documentação técnica

O objetivo é construir um ambiente funcional, documentado e reproduzível, utilizando ferramentas amplamente encontradas em ambientes corporativos.

---

## 🎯 Objetivos

### Objetivo principal

Construir e documentar uma infraestrutura corporativa de pequeno porte utilizando virtualização e tecnologias open source.

### Objetivos técnicos

- Implementar um controlador de domínio baseado em Samba Active Directory
- Implementar DNS integrado ao ambiente
- Implementar serviços de rede
- Criar servidor de arquivos utilizando SMB
- Integrar estações Windows ao domínio
- Integrar estações Linux ao domínio
- Implementar monitoramento com Zabbix
- Utilizar Grafana para visualização de métricas
- Criar scripts para tarefas administrativas
- Simular falhas e incidentes
- Aplicar metodologia de troubleshooting
- Documentar procedimentos e soluções

---

# 🏗️ Arquitetura

A arquitetura planejada para o laboratório é baseada em uma rede corporativa virtualizada.

```text
                         INTERNET
                            │
                     ┌──────▼──────┐
                     │   FIREWALL  │
                     │  OPNsense   │
                     └──────┬──────┘
                            │
                     10.10.10.0/24
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   ┌────▼─────┐        ┌────▼─────┐       ┌────▼─────┐
   │   DC01   │        │   SRV01   │       │   MON01  │
   │  Debian  │        │  Debian   │       │  Debian  │
   │ Samba AD │        │ FileServer│       │  Zabbix  │
   │   DNS    │        │    SMB    │       │  Grafana │
   └──────────┘        └───────────┘       └──────────┘
        │
        │
        │       CORPORATE DOMAIN
        │          CORP.LAB
        │
   ┌────┴───────────────┐
   │                    │
┌──▼─────┐          ┌───▼──────┐
│ WKS01  │          │  WKS02   │
│Windows │          │  Debian  │
└────────┘          └──────────┘