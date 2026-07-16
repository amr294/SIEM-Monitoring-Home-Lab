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
- Splunk Enterprise (Planned)

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

## Documentation

Detailed implementation guides will be added throughout the project.

- Lab Planning
- VMware Installation
- Windows Server Installation
- Active Directory Configuration
- Organizational Units
- Windows 11 Domain Join
- Sysmon Deployment
- Wazuh Deployment
- Attack Simulation
- Detection Engineering

---

## Project Progress

- [x] VMware Workstation installed
- [x] Windows Server 2022 deployed
- [x] Static IP configured
- [x] Active Directory Domain Services installed
- [x] Domain Controller promoted
- [ ] Organizational Units (OUs)
- [ ] User and Group creation
- [ ] Windows 11 domain join
- [ ] Sysmon deployment
- [ ] Wazuh deployment
- [ ] Attack simulation
- [ ] Detection rules

## Disclaimer

This lab is built for educational purposes only. All attacks, detections, and security testing are performed in an isolated virtual environment.