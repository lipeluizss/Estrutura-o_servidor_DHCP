# Projeto de Reestruturação de Rede - SEPLAN/RN

- [Visão Geral](#visão-geral)
- [Motivação](#motivação)
- [Arquitetura Planejada](#arquitetura-planejada)
- [Distribuição de Endereços IP](#distribuição-de-endereços-ip)
- [Implementação Gradual](#implementação-gradual)
- [Configuração DHCP](#configuração-dhcp)
- [Boas Práticas](#boas-práticas)
- [Troubleshooting](#troubleshooting)

---

##  Visão Geral

Este documento descreve o projeto de reestruturação da rede da Secretaria de Planejamento do Rio Grande do Norte (SEPLAN/RN), com foco na organização lógica de endereços IP por setor administrativo, fazendo a separação por subdominio ligado a cada OU(Unidade Organizacional).

### Informações Técnicas

- **Rede:** 10.3.X.X.0/22 (1024 endereços disponíveis)
- **Domínio:** Dominio.local
- **Controlador de Domínio:** Samba4 AD DC
- **Servidor DHCP:**  (10.X.X.131)
- **Gateway:** 10.X.X.254
- **Servidor de monitoramento**  (10.X.X.126)
- **Servidor de arquivos**  (10.X.X.112)
- **DNS Principal:** 10.X.X.255 

---

### Problemas Identificados

1. **Desorganização**: IPs distribuídos sem critério lógico por setor
2. **Dificuldade de gestão**: Impossível identificar rapidamente a que setor pertence um IP
3. **Problemas de segurança**: Dificuldade em aplicar políticas de firewall segmentadas
4. **Crescimento desordenado**: Falta de planejamento para expansão futura

### Objetivos

1. Organizar IPs por departamento/setor
2. Facilitar identificação de origem de problemas de rede
3. Permitir aplicação de políticas (firewall, GPO) por setor
4. Preparar a rede para futura implementação de VLANs
5. Reservar espaços para crescimento controlado

---

### Estrutura Organizacional

seplan.govrn (Domínio AD)
└── Secretaria
├── Infraestrutura (Servidores, Switches, Roteadores)
└── Setores
├── SETOR A  - 10 IPs
├── SETOR B  - 15 IPs
├── SETOR C  - 10 IPs
├── SETOR D  - 10 IPs
├── SETOR E  - 10 IPs
├── SETOR F  - 20 IPs
├── SETOR G  - 30 IPs
├── SETOR H  - 20 IPs
├── SETOR I  - 20 IPs
├── SETOR J  - 10 IPs
├── SETOR K  - 5 IPs
├── SETOR L  - 10 IPs
├── SETOR M  - 10 IPs
├── SETOR N  - 15 IPs
└── SETOR O  - 20 IPs

---

### Critérios de Distribuição

- **30 IPs iniciais**: Reservados para infraestrutura crítica (servidores, switches, roteadores)
- **Ranges personalizados por setor**: De 5 a 30 IPs conforme necessidade
- **15 IPs de lacuna**: Espaço entre setores para expansão futura e separação lógica
- **Total de setores**: 15

---

##  Distribuição de Endereços IP

### Infraestrutura (10.X.X.1 - 10.X.X.46)

---

### Setores ( ~ 20 IPs + 15 IPs de lacuna)

### Setores (Ranges personalizados + 15 IPs de lacuna)

| Setor | Faixa de IPs | Quantidade | Lacuna |
|-------|--------------|------------|--------|
| **A** | 10.X.X.46 - 10.X.X.56 | 10 IPs | 10.X.X.57 - 10.X.X.59 |
| **B** | 10.X.X.60 - 10.X.X.75 | 15 IPs | 10.X.X.76 - 10.X.X.85 |
| **C** | 10.X.X.86 - 10.X.X.95 | 10 IPs | 10.X.X.96 - 10.X.X.110 |
| **D** | 10.X.X.111 - 10.X.X.120 | 10 IPs | 10.X.X.121 - 10.X.X.135 |
| **E** | 10.X.X.136 - 10.X.X.145 | 10 IPs | 10.X.X.146 - 10.X.X.160 |
| **F** | 10.X.X.161 - 10.X.X.180 | 20 IPs | 10.X.X.181 - 10.X.X.195 |
| **G** | 10.X.X.196 - 10.X.X.225 | 30 IPs | 10.X.X.226 - 10.X.X.240 |
| **H** | 10.X.X.241 - 10.X.X.260 | 20 IPs | 10.X.X.261 - 10.X.X.275 |
| **I** | 10.X.X.1 - 10.X.X.20 | 20 IPs | 10.X.X.21 - 10.X.X.35 |
| **J** | 10.X.X.36 - 10.X.X.45 | 10 IPs | 10.X.X.46 - 10.X.X.60 |
| **K** | 10.X.X.61 - 10.X.X.65 | 5 IPs | 10.X.X.66 - 10.X.X.80 |
| **L** | 10.X.X.81 - 10.X.X.90 | 10 IPs | 10.X.X.91 - 10.X.X.105 |
| **M** | 10.X.X.106 - 10.X.X.115 | 10 IPs | 10.X.X.116 - 10.X.X.130 |
| **N** | 10.X.X.131 - 10.X.X.145 | 15 IPs | 10.X.X.146 - 10.X.X.160 |
| **O** | 10.X.X.161 - 10.X.X.180 | 20 IPs | 10.X.X.181 - ... |

### Resumo Estatístico

- **Infraestrutura**: 30 IPs
- **Setores**: 199 IPs totais (ranges variados)
- **Lacunas**: 209 IPs (14 × 15)
- **Total planejado**: ~435 IPs
- **Disponível para expansão**: ~588 IPs

---

##  Implementação Gradual

### Por que Implementação Gradual?

- Minimiza riscos de downtime
- Permite testes e validação por etapa e por setor
- Evita sobrecarga de trabalho
- Facilita rollback em caso de problemas
- **Não requer mudança de IPs de servidores existentes**

---

- [ISC DHCP Server Documentation](https://kb.isc.org/docs/isc-dhcp-44-manual-pages-dhcpdconf)
- [Samba4 AD DC Wiki](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)
- [RFC 2131 - DHCP](https://datatracker.ietf.org/doc/html/rfc2131)
