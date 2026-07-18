# 04 - Network Configuration and VMware Troubleshooting

## Overview

This phase focuses on configuring reliable networking for the Windows Server virtual machine before promoting it to a Domain Controller.

The process included installing VMware Tools, assigning a static IPv4 address, validating network connectivity, and troubleshooting an unexpected VMware networking issue that interrupted Internet access.

The initial configuration consisted of:

- Installing VMware Tools
- Verifying network adapter functionality
- Assigning a static IPv4 address
- Testing connectivity

Although the network configuration appeared correct, the server unexpectedly lost Internet connectivity after switching from DHCP to a static IP configuration. This document walks through the troubleshooting process used to identify and resolve the issue.

---

# Step 1 — Install VMware Tools

Installing VMware Tools improves virtual hardware compatibility and ensures that the guest operating system can properly interact with VMware's virtual devices.

![Install VMware Tools](../images/04-network-configuration-and-troubleshooting/01-install-vmware-tools.png)

After installation, the server booted normally and all VMware drivers were successfully loaded.

![Windows Server Login](../images/04-network-configuration-and-troubleshooting/02-windows-server-login.png)

---

# Step 2 — Configure Network Settings

The server initially received an IP address from VMware's NAT DHCP service.

![DHCP Configuration](../images/04-network-configuration-and-troubleshooting/04-dhcp-ipconfig.png)

To prepare the server for Active Directory deployment, a static IPv4 configuration was assigned.

Configuration:

| Setting | Value |
|---------|-------|
| IP Address | 192.168.247.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.247.2 |
| Preferred DNS | 8.8.8.8 *(temporary)* |

Google Public DNS (8.8.8.8) was used temporarily to validate external name resolution during network troubleshooting. It would later be replaced with the local Domain Controller's DNS service after Active Directory installation.

![Static IPv4 Configuration](../images/04-network-configuration-and-troubleshooting/05-configure-static-ipv4.png)

---

# Step 3 — Verify Network Configuration

The routing table and interface configuration appeared correct after applying the static address.

![Route Table](../images/04-network-configuration-and-troubleshooting/06-route-print-initial.png)

Additional verification confirmed that the server interface was configured as expected.

![IP Configuration](../images/04-network-configuration-and-troubleshooting/09-server-ipconfig-all.png)

At this stage, there were no obvious configuration errors.

---

# Step 4 — Connectivity Failure

Despite the correct IPv4 configuration, the server was unable to communicate with the default gateway and external networks.

![Ping Failure](../images/04-network-configuration-and-troubleshooting/07-ping-host-failure.png)

Since the IP configuration appeared correct, the issue was unlikely to be caused by an incorrect static address.

---

# Step 5 — Windows Network Diagnostics

To isolate the problem, several Windows networking tools were used:

- `ipconfig`
- `route print`
- `ping`
- `netstat`
- `arp`

These commands were used to verify interface configuration, routing information, and local network communication.

![Windows Diagnostics](../images/04-network-configuration-and-troubleshooting/11-network-diagnostics.png)

Host-side diagnostics were also performed to verify communication between the VMware host and guest.

![Host Diagnostics](../images/04-network-configuration-and-troubleshooting/12-host-network-diagnostics.png)

The collected information suggested that Windows networking itself was functioning correctly.

---

# Step 6 — VMware Network Investigation

The investigation then shifted toward VMware's virtual networking components.

First, the VMware NAT service was verified to ensure it was running correctly.

![VMware NAT Service](../images/04-network-configuration-and-troubleshooting/14-vmware-nat-service-running.png)

Network sockets associated with VMware NAT were also inspected.

![VMware Netstat](../images/04-network-configuration-and-troubleshooting/15-netstat-vmnet8.png)

Finally, the VMware Virtual Network Editor was reviewed to verify the VMnet8 NAT configuration.

![Virtual Network Editor](../images/04-network-configuration-and-troubleshooting/24-virtual-network-editor-main.png)

![VMnet8 NAT Settings](../images/04-network-configuration-and-troubleshooting/25-vmnet8-nat-settings.png)

Although the configuration appeared normal, connectivity problems persisted.

---

# Step 7 — Root Cause and Resolution

After verifying:

- Windows TCP/IP configuration
- Routing table
- VMware NAT service
- Virtual network settings

the issue was isolated to VMware's virtual networking configuration.

The VMware Virtual Network Editor's **Restore Defaults** option was used to recreate the default virtual networks (VMnet0, VMnet1, and VMnet8).

![Restore VMware Defaults](../images/04-network-configuration-and-troubleshooting/27-restore-vmware-network-defaults.png)

Restoring VMware's default virtual networks immediately restored network connectivity, confirming that the issue originated within VMware's virtual networking configuration rather than the Windows Server guest.

---

# Step 8 — Recovery Verification

After restoring VMware's default networking configuration, the server successfully received a new DHCP lease from the recreated VMnet8 network.

![DHCP After Reset](../images/04-network-configuration-and-troubleshooting/28-dhcp-after-network-reset.png)

With connectivity restored, the server was configured once again with its final static IPv4 address.

![Final Static Configuration](../images/04-network-configuration-and-troubleshooting/29-configure-final-static-ip.png)

Connectivity tests confirmed successful communication with both the gateway and external networks.

![Google Ping Success](../images/04-network-configuration-and-troubleshooting/30-google-ping-success.png)

---

# Step 9 — Final Validation

A final review confirmed that the server was operating with the expected network configuration.

![Server Manager Verification](../images/04-network-configuration-and-troubleshooting/31-server-manager-network-verification.png)

The server was then renamed to **DC01**, completing the network preparation stage before Active Directory installation.

![Rename Server](../images/04-network-configuration-and-troubleshooting/32-rename-server-dc01.png)

---

# Lessons Learned

This troubleshooting exercise reinforced several important concepts:

- Always verify the operating system configuration before assuming virtualization issues.
- Use multiple diagnostic tools (`ipconfig`, `route`, `ping`, `arp`, and `netstat`) to isolate network problems.
- VMware networking can become corrupted even when the configuration appears correct.
- Restoring VMware's default virtual networks can resolve complex NAT networking issues.
- Systematic troubleshooting is often more valuable than simply applying configuration changes.

---

## Outcome

At the end of this phase, the Windows Server virtual machine had:

- VMware Tools installed
- Reliable NAT networking
- Functional Internet connectivity
- A verified static IPv4 configuration
- A renamed server (**DC01**)

With networking fully restored and validated, the environment was ready for Active Directory Domain Services (AD DS) installation in the next phase of the project.