# Lab Planning

## Objective

The goal of this project is to build a realistic Security Information and Event Management (SIEM) home lab that simulates a small enterprise environment.

The lab will be used to:

- Learn Active Directory administration
- Deploy and configure Windows infrastructure
- Generate Windows security events
- Collect and analyze logs
- Simulate attacker activity
- Develop detection capabilities
- Improve incident investigation skills

---

## Lab Architecture

The following diagram illustrates the overall architecture of the lab environment, including the virtualization platform, network topology, Windows infrastructure, attacker machine, and SIEM platform.

![SIEM Home Lab Architecture](../images/architecture/lab-architecture.png)

---

## Lab Components

| Component | Purpose |
|-----------|---------|
| VMware Workstation Pro | Virtualization platform |
| Windows Server 2022 | Domain Controller |
| Windows 11 | Domain-joined client |
| Kali Linux | Attacker machine |
| Wazuh | SIEM platform |
| Sysmon | Advanced Windows logging |

---

## Planned Network

| Machine | Role |
|----------|------|
| DC01 | Domain Controller |
| WIN11 | Domain User Workstation |
| KALI | Attacker Machine |
| WAZUH | Wazuh Manager |

---

## Learning Objectives

Throughout this project I aim to strengthen my understanding of:

- Active Directory
- DNS
- Windows Authentication
- Windows Event Logs
- Sysmon
- Wazuh
- SIEM Operations
- Threat Detection
- Incident Investigation
- PowerShell
- Git & GitHub Documentation

---

## Expected Outcome

By the end of this project, the lab will be capable of generating Windows security events, forwarding them to a SIEM platform, and detecting simulated attacker activity using custom detection rules.