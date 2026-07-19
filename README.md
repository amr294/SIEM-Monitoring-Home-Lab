# SIEM Monitoring Home Lab

## Project Overview

This project documents the design, implementation, and testing of a Security Information and Event Management (SIEM) home lab.

The lab simulates a small enterprise environment by deploying Windows infrastructure, generating security events, collecting logs, and analyzing them using security monitoring tools.

This repository contains the complete project documentation, configuration steps, screenshots, lessons learned, and future improvements throughout the project.

## Technologies

- VMware Workstation Pro
- Windows Server 2022
- Windows 11
- Kali Linux
- Active Directory Domain Services (AD DS)
- DNS
- PowerShell
- Sysmon
- Wazuh

## Project Goals

- Build an enterprise-style Active Directory environment.
- Configure Windows infrastructure and networking.
- Generate realistic Windows security events.
- Collect and analyze security logs using SIEM platforms.
- Detect attacker activity using security analytics.
- Simulate adversary techniques in a controlled environment.
- Document every implementation step.

## Lab Architecture

The home lab consists of:

- Windows Server 2022 (Domain Controller)
- Windows 11 Client
- Kali Linux Attacker Machine
- Wazuh SIEM Server
- VMware Workstation Pro virtual network

## Skills Demonstrated

- Active Directory Administration
- Windows Server Administration
- DNS Configuration
- Virtualization
- Network Configuration
- Security Monitoring
- SIEM Deployment
- Windows Event Logging
- Documentation
- Git & GitHub

## 📚 Documentation

The lab is documented step by step, covering the complete deployment process from infrastructure planning to security monitoring. Each document includes configuration details, screenshots, validation steps, and troubleshooting notes where applicable.

| Step | Documentation | Description |
|------|---------------|-------------|
| 01 | [Lab Planning](docs/01-Lab-Planning.md) | Define the project scope, objectives, and lab architecture. |
| 02 | [VMware Setup](docs/02-VMware-Setup.md) | Create the virtual networking environment and virtual machines. |
| 03 | [Windows Server Installation](docs/03-Windows-Server-Installation.md) | Install Windows Server 2022 and perform the initial system configuration. |
| 04 | [Network Configuration and VMware Troubleshooting](docs/04-Network-Configuration-and-Troubleshooting.md) | Configure networking, troubleshoot connectivity issues, and validate VMware NAT functionality. |
| 05 | [Active Directory Domain Services (AD DS) Installation](docs/05-Active-Directory-Domain-Services-Installation.md) | Install the AD DS role, create the first forest, and promote the server to a Domain Controller. |
| 06 | *Coming Soon* | Organizational Units and User Management |
| 07 | *Coming Soon* | Group Policy Configuration |
| 08 | *Coming Soon* | Windows 11 Domain Join |
| 09 | *Coming Soon* | Sysmon Deployment |
| 10 | *Coming Soon* | Wazuh SIEM Deployment |
| 11 | *Coming Soon* | Attack Simulation |
| 12 | *Coming Soon* | Detection and Analysis |

---

## Project Progress

- [x] VMware Workstation installed
- [x] Windows Server 2022 deployed
- [x] Static IP configured
- [x] Active Directory Domain Services (AD DS) installation
- [x] Domain Controller promotion
- [x] DNS installation and integration
- [ ] Organizational Units (OUs)
- [ ] User and Group creation
- [ ] Windows 11 domain join
- [ ] Sysmon deployment
- [ ] Wazuh deployment
- [ ] Attack simulation
- [ ] Detection rules

## Disclaimer

This lab is built for educational purposes only. All attacks, detections, and security testing are performed in an isolated virtual environment.