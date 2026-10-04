# IT Support Practice Lab

## Lab Overview

This lab is a small copy of a real company network and server system. It uses VMware to help you practice the work of an IT Support Analyst. The system has a Domain Controller, a Client Workstation, and a ticket server (osTicket). You will create Virtual Machines and manage Active Directory, DNS, and DHCP. You will also set up Group Policy, control file access with NTFS and Share Permissions, and manage updates (patches) with WSUS. Then you will solve problems (troubleshooting) in Ticket Scenarios, which are based on real work steps.

---




# Lab Environment



## 1. Virtual Machine Configuration

| VM | Role | OS | vCPU | RAM | Virtual Disk | Network | IP / Addressing |
|---|---|---|---:|---:|---|---|---|
| **LUNA-DC01** | Domain Controller / Core Infrastructure | Windows Server 2025 Desktop Experience | 2 | 4 GB | 64 GB OS + 80 GB WSUS Data | VMnet8 NAT | `192.168.10.10` Static |
| **LUNA-W11-01** | Domain-Joined Workstation / Client | Windows 11 Enterprise | 2 | 4 GB | 64 GB | VMnet8 NAT | DHCP `192.168.10.100–200` |
| **LUNA-DEB01** | osTicket / Docker / MariaDB Host | Debian Linux | 2 | 4 GB | 40 GB | VMnet8 NAT | `192.168.10.20` DHCP Reservation |

---

# Network Configuration

| Setting | Lab Value |
|---|---|
| Network | `192.168.10.0/24` |
| Subnet Mask | `255.255.255.0` |
| VMware Network | `VMnet8` |
| Network Mode | NAT |
| DHCP Pool | `192.168.10.100–192.168.10.200` |
| Domain Controller IP | `192.168.10.10` |
| Debian Reserved IP | `192.168.10.20` |
| Windows Client Addressing | DHCP |
| Domain | `luna.local` |

### Addressing Model

The IP plan is simple on purpose:

```text
192.168.10.10  → LUNA-DC01
                 Static IP

192.168.10.20  → LUNA-DEB01
                 DHCP Reservation

192.168.10.100
      ↓
192.168.10.200  → Client DHCP Range
```

`LUNA-DC01` uses a static IP address because main services need an address that does not change.

`LUNA-DEB01` uses a DHCP Reservation. The DHCP server always gives the same address (`192.168.10.20`) to the Debian VM, and we still manage the address in one place.

`LUNA-W11-01` gets its address from DHCP automatically.

---

# VM Specification Details

## LUNA-DC01

| Setting | Value |
|---|---|
| Hostname | `LUNA-DC01` |
| Role | Domain Controller / Infrastructure Server |
| OS | Windows Server 2025 Desktop Experience |
| vCPU | 2 |
| RAM | 4 GB |
| OS Disk | 64 GB |
| WSUS Data Disk | 80 GB |
| Disk Provisioning | Thin Provisioned |
| Network Adapter | VMnet8 NAT |
| IP Address | `192.168.10.10` |
| Addressing | Static |
| Domain | `luna.local` |

### Services

```text
AD DS
DNS
DHCP
WSUS
```

### Main Jobs

```text
Active Directory Domain Services
        ↓
Users, Login, and Domain

DNS
        ↓
Name Lookup

DHCP
        ↓
IP Address Assignment

WSUS
        ↓
Windows Update Management
```

---

## LUNA-W11-01

| Setting | Value |
|---|---|
| Hostname | `LUNA-W11-01` |
| Role | Domain-Joined Workstation / Client |
| OS | Windows 11 Enterprise |
| vCPU | 2 |
| RAM | 4 GB |
| Disk | 64 GB |
| Disk Provisioning | Thin Provisioned |
| Network Adapter | VMnet8 NAT |
| IP Addressing | DHCP |
| DHCP Range | `192.168.10.100–200` |
| Domain | `luna.local` |

### Windows 11 Virtual Machine Needs

```text
UEFI
Secure Boot
vTPM 2.0
```

### Main Jobs

```text
User Computer
        ↓
Domain Join
        ↓
Login
        ↓
Group Policy
        ↓
Windows Update / WSUS
        ↓
IT Support Troubleshooting
```

---

## LUNA-DEB01

| Setting | Value |
|---|---|
| Hostname | `LUNA-DEB01` |
| Role | IT Support Application Server |
| OS | Debian Linux |
| vCPU | 2 |
| RAM | 4 GB |
| Disk | 40 GB |
| Disk Provisioning | Thin Provisioned |
| Network Adapter | VMnet8 NAT |
| IP Address | `192.168.10.20` |
| Addressing | DHCP Reservation |

### Services

```text
Docker
MariaDB
osTicket
```

### Application Stack

```text
LUNA-DEB01
     │
     ▼
   Docker
     │
     ├──────────────► MariaDB
     │
     └──────────────► osTicket
                           │
                           ▼
                       Port 8081
```

---

# Service Distribution

| Service | Host | Purpose |
|---|---|---|
| AD DS | `LUNA-DC01` | Domain, users, and login |
| DNS | `LUNA-DC01` | Name lookup inside the network |
| DHCP | `LUNA-DC01` | Gives IP addresses automatically |
| WSUS | `LUNA-DC01` | Manages Windows updates |
| Group Policy | `LUNA-DC01` | Sets up workstations from one place |
| Domain-Joined Client | `LUNA-W11-01` | Test user computer |
| Docker | `LUNA-DEB01` | Container platform |
| MariaDB | `LUNA-DEB01` | Database for osTicket |
| osTicket | `LUNA-DEB01` | IT support ticket system |

---

# Storage Design

We use separate virtual disks for different jobs.

### LUNA-DC01

```text
Disk 1 — 64 GB
→ Windows Server 2025
→ OS / System Files
→ Active Directory
→ DNS
→ DHCP
→ General Server Data

Disk 2 — 80 GB
→ WSUS Data / Update Files
```

The WSUS disk is separate. This keeps update files away from the OS disk. It is also easier to understand and manage.

### LUNA-W11-01

```text
Disk 1 — 64 GB
→ Windows 11 Enterprise
→ User / Endpoint Lab
```

### LUNA-DEB01

```text
Disk 1 — 40 GB
→ Debian
→ Docker
→ MariaDB
→ osTicket
```

All virtual disks use **Thin Provisioning**. This means a disk does not take all of its space on the physical host at the start.

---
# General Settings
These values are fixed for the lab.

| Category | Final Value |
|---|---|
| Hypervisor | VMware |
| Network | VMnet8 NAT |
| Lab Subnet | `192.168.10.0/24` |
| Domain | `luna.local` |
| Disk Provisioning | Thin Provisioned |

## Virtual Machine Settings

| Item | `LUNA-DC01` | `LUNA-W11-01` | `LUNA-DEB01` |
|---|---|---|---|
| Role | Domain Controller | Windows Client | Debian Server |
| OS | Windows Server 2025 Desktop Experience | Windows 11 Enterprise | Debian Linux |
| vCPU | 2 | 2 | 2 |
| RAM | 4 GB | 4 GB | 4 GB |
| OS Disk | 64 GB | 64 GB | 40 GB |
| WSUS Disk | 80 GB | - | - |
| IP Address | `192.168.10.10` | `192.168.10.100–200` | `192.168.10.20` |
| Addressing | Static | DHCP | DHCP Reservation |

---

# Lab Architecture Summary

```text
                         IT SUPPORT PRACTICE LAB

                         LAB NETWORK
                       192.168.10.0/24
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
   │ LUNA-DC01   │     │ LUNA-W11-01 │     │ LUNA-DEB01  │
   │             │     │             │     │             │
   │ .10 Static  │     │ DHCP        │     │ .20 Reserve │
   │             │     │             │     │             │
   │ Win Server  │     │ Win 11 Ent. │     │ Debian      │
   │ 2025        │     │             │     │             │
   └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
          │                   │                   │
          ▼                   │                   ▼
   ┌───────────────┐          │            ┌───────────────┐
   │ Infrastructure│          │            │ Application   │
   │               │          │            │ Stack         │
   │ AD DS         │◄─────────┤            │ Docker        │
   │ DNS           │          │            │ MariaDB       │
   │ DHCP          │          │            │ osTicket      │
   │ WSUS          │          │            │ :8081         │
   └───────────────┘          │            └───────────────┘
                              ▼
                     ┌──────────────────┐
                     │ Domain-Joined    │
                     │ Windows Endpoint │
                     └──────────────────┘
```

---
# Initial Server Setup

## Server Details
- **Hostname:** LUNA-DC01
- **OS:** Windows Server 2025
- **Role:** Domain Controller
- **Static IP:** 192.168.10.10/24
- **Gateway:** 192.168.10.1
- **DNS:** 127.0.0.1

## Steps Performed

