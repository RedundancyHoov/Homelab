#  Homelab Upgrade 3400  
 Multi-Vendor Network Engineering Lab • Cisco • Juniper • VMware • Linux • CCNA/RHCSA

This repo documents my homelab — the environment I use to level up as a network engineer while studying for the **CCNA** and **RHCSA**. It’s built like a real environment: routing, switching, VLAN design, VMware networking, segmentation, security, and automation across Cisco + Juniper gear.

The goal: treat this like a production-style lab where I can design, break, fix, and document everything.

---

## 🔧 Core Hardware

### 🌐 Networking
- **Juniper EX3400-48P**  
  - Core Layer 3 switch  
  - SVIs, inter-VLAN routing, static routes  
  - Management in **VLAN 99 (172.16.99.x)**

- **Cisco 1921 Router**  
  - WAN edge to ISP router  
  - NAT, ACLs, default gateway for the lab  
  - WAN: `192.168.86.x` (ISP side, masked)  
  - LAN / Lab side: `10.0.0.x`

- **Cisco 2960**  
  - Access switch  
  - Trunks to EX3400  
  - Used for wired endpoints, lab gear, and AP uplinks

### 🖥 Compute / Virtualization
- **HP Mini PC**  
  - Runs **VMware Workstation Pro**  
  - Additional USB 3.0 → RJ45 NIC for trunk/access into lab VLANs  
  - VMs live primarily in **VLAN 10 (VMs)** and **VLAN 20 (TEST)**

- **Dell R630 / R240**  
  - Future expansion for hypervisor cluster, Proxmox, etc.

 Support / Services
- **Raspberry Pi 4**  
  - TFTP / backup server  
  - Used for automated config backups of routers/switches

---

## 🏗Network Architecture

### 🔹 VLAN & Subnet Scheme

All VLAN subnets follow this pattern:

> **172.16.x.x


| VLAN | Name  | Purpose                                               | Subnet (Masked)   |
|------|-------|-------------------------------------------------------|-------------------|
| 10   | VMs   | Virtual machines, servers, internal services          | `172.16.10.x`     |
| 20   | TEST  | Testing, break/fix, isolated experiments              | `172.16.20.x`     |
| 30   | LAB   | Lab devices, workstations, engineering tools          | `172.16.30.x`     |
| 40   | WIFI  | Wireless clients, separated from wired lab networks   | `172.16.40.x`     |
| 99   | MGMT  | Management (switches, router, infra devices)          | `172.16.99.x`     |

###  Layer 3 Design

- **Juniper EX3400 48P**  
  - Hosts all SVIs for VLANs 10, 20, 30, 40, and 99  
  - Performs inter-VLAN routing inside the lab  
- **Cisco 1921**  
  - Default route out to ISP  
  - NAT overload for lab → internet  
  - Static/connected route back to `172.16.10.x/20.x/30.x/40.x/99.x` via EX3400
- **MGMT (VLAN 99)**  
  - Dedicated for management IPs (switches, router, other infra)  
  - No user endpoints here

---

##  VMware Workstation Networking

VMware Workstation on the HP Mini is tied into the lab like a real data center segment.

- VMs primarily use:
  - **VLAN 10 → `172.16.10.x`** for VMs/services  
  - **VLAN 20 → `172.16.20.x`** for testing/break-fix environments
- USB NIC is used as:
  - Either a trunk or specific access port into required VLANs
- VMnet mappings and diagrams are stored in the `/vmware` folder
- All IPs in documentation are masked as `x` in the last octet

Troubleshooting covered in this lab includes:

- Default gateway issues (`no route to host`, gateway unreachable)
- ACLs blocking VM ↔ LAN or VM ↔ internet
- Incorrect VMware network mappings (bridged vs host-only vs NAT)
- MTU and trunk/access misconfigurations between VMware and switches

---

## 📁 Repo Structure

Planned structure for this repo:

```text
/configs
  /cisco
    1921-running-config.txt
    2960-access-config.txt
  /juniper
    ex3400-switch-options.conf
    ex3400-vlans.conf
    ex3400-routing.conf

/vmware
  vm-network-diagrams/
  vm-notes.md

/diagrams
  homelab-topology.drawio
  vlan-layout.drawio

/documentation
  readme-images/
  homelab-overview.md
  rack-setup.md
