# VMware Workstation Setup

## Objective

Prepare the virtualization environment that will host the entire SIEM home lab.

---

## Software

- VMware Workstation Pro
- Windows 11 Host Machine

---

## Virtual Machines

| VM | Operating System | Purpose |
|----|------------------|---------|
| DC01 | Windows Server 2022 | Active Directory Domain Controller |
| WIN11 | Windows 11 | Domain Client |
| KALI | Kali Linux | Attacker Machine |
| WAZUH *(Planned)* | Ubuntu Server | SIEM Platform |

---

## Network Configuration

The virtual machines communicate using VMware virtual networking.

All machines are connected to the same isolated virtual network to simulate an internal enterprise environment.

---

## Resource Allocation

| VM | CPU | RAM |
|----|-----|-----|
| DC01 | 2 vCPU | 4 GB |
| WIN11 | 2 vCPU | 4 GB |
| KALI | 2 vCPU | 4 GB |
| WAZUH | 4 vCPU | 8 GB *(planned)* |

---

## Progress

- VMware Workstation installed
- Windows Server virtual machine created
- Windows 11 virtual machine created
- Kali Linux virtual machine created
- Virtual networking configured

---

## Lessons Learned

- VMware networking is critical for communication between virtual machines.
- Resource allocation should balance host performance and lab requirements.
- Proper VM naming improves project organization.