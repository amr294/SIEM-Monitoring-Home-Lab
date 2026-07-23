# Wazuh Agent Deployment

## Overview

After completing the deployment of the Wazuh SIEM platform, the next phase of the project focused on onboarding Windows endpoints into the monitoring environment. This stage involved deploying the Wazuh agent to the Domain Controller (DC01) and the Windows 11 client, establishing secure communication with the Wazuh Manager, validating endpoint registration, and confirming that Windows Security and Sysmon telemetry were successfully ingested into the SIEM.

Unlike the server deployment, this phase concentrated on endpoint integration and telemetry validation rather than infrastructure installation. Successful completion of this stage established centralized monitoring across the Windows environment and prepared the lab for attack simulation and detection engineering.

---

## Deployment Objectives

The objectives of this phase were to:

- Deploy the Wazuh agent on Windows endpoints.
- Register both endpoints with the Wazuh Manager.
- Validate secure communication between the agents and the SIEM platform.
- Verify Windows Security event collection.
- Validate Sysmon Operational event ingestion.
- Confirm complete endpoint visibility within the Wazuh Dashboard.

---

## Existing Infrastructure

Before deploying the endpoint agents, the SIEM infrastructure had already been fully implemented and validated.

The environment included:

- Native Ubuntu Server hosting the Wazuh platform
- Wazuh Manager
- Wazuh Indexer
- Filebeat
- Wazuh Dashboard
- Active Directory Domain Controller (DC01)
- Windows 11 domain workstation
- Sysmon deployed on both Windows systems

With the SIEM infrastructure operational, endpoint onboarding could proceed.

---

# 4. Deploying the Domain Controller Agent

## 4.1 Generating the Installation Command

The deployment process began from the **Endpoints** section of the Wazuh Dashboard.

Using the built-in deployment wizard, the Windows MSI package was selected, the Wazuh Manager address was specified, and the agent name was configured. The dashboard automatically generated a PowerShell installation command containing the required configuration parameters.

<p align="center">
  <img src="../images/agent deployment/01-wazuh-deploy-new-agent-page.png" width="900">
</p>

<p align="center">
  <img src="../images/agent deployment/02-wazuh-agent-deployment-options.png" width="900">
</p>

<p align="center">
  <img src="../images/agent deployment/03-generated-agent-install-command.png" width="900">
</p>

---

## 4.2 Installing the Agent

The generated installation command was executed from an elevated PowerShell session on the Domain Controller.

The command downloaded the Windows MSI package, installed the Wazuh Agent silently, configured the Manager address, and assigned the endpoint name during installation.

<p align="center">
  <img src="../images/agent deployment/05-dc01-agent-download-progress.png" width="900">
</p>

---

## 4.3 Starting the Agent Service

After installation completed, the Wazuh service was verified to ensure the agent had been installed correctly.

The service was then started to establish communication with the Wazuh Manager.

<p align="center">
  <img src="../images/agent deployment/06-dc01-wazuh-service-running.png" width="900">
</p>

---

## 4.4 Validating Endpoint Registration

Once the service was running, the Domain Controller successfully registered with the Wazuh Manager.

The endpoint appeared within the Dashboard as an active managed agent.

<p align="center">
  <img src="../images/agent deployment/07-dc01-agent-registered-on-manager.png" width="900">
</p>

Initial Windows Security events were immediately visible, confirming successful communication between the endpoint and the SIEM platform.

<p align="center">
  <img src="../images/agent deployment/08-dc01-events-received.png" width="900">
</p>

---

# 5. Troubleshooting Sysmon Telemetry

## 5.1 Initial Validation

Although Windows Security events were successfully collected, filtering specifically for Microsoft Sysmon events initially returned no results.

This indicated that endpoint communication was functioning correctly, while Sysmon telemetry required additional validation.

<p align="center">
  <img src="../images/agent deployment/09-dc01-sysmon-search-no-results.png" width="900">
</p>

---

## 5.2 Reviewing the Agent Configuration

During troubleshooting, the Wazuh Agent configuration file (`ossec.conf`) was inspected.

Initially, the configuration file could not be opened due to insufficient permissions.

<p align="center">
  <img src="../images/agent deployment/10-ossec-config-permission-denied.png" width="900">
</p>

After opening the editor with administrative privileges, the configuration was successfully reviewed.

<p align="center">
  <img src="../images/agent deployment/11-ossec-config-sysmon-enabled.png" width="900">
</p>

The configuration confirmed that the Microsoft-Windows-Sysmon/Operational channel was included in the monitored event sources.

---

## 5.3 Validating Sysmon Collection

After validating the agent configuration, Sysmon Operational events became visible within the Wazuh Dashboard.

This confirmed successful collection and forwarding of endpoint telemetry from Sysmon into the SIEM platform.

<p align="center">
  <img src="../images/agent deployment/12-dc01-sysmon-events-ingested.png" width="900">
</p>

---

# 6. Deploying the Windows 11 Client Agent

Following successful validation of the Domain Controller, the same deployment procedure was repeated for the Windows 11 workstation.

The deployment wizard generated a Windows-specific installation command configured for the second endpoint.

<p align="center">
  <img src="../images/agent deployment/13-win11-generated-install-command.png" width="900">
</p>

The command was executed from an elevated PowerShell session, installing and configuring the Wazuh Agent.

<p align="center">
  <img src="../images/agent deployment/14-win11-agent-download-progress.png" width="900">
</p>

The Windows 11 endpoint successfully connected to the Wazuh Manager and immediately began forwarding security telemetry.

---

# 7. Platform Validation

With both endpoints deployed, the Wazuh Dashboard confirmed that the Domain Controller and Windows 11 workstation were successfully registered and actively communicating with the Manager.

<p align="center">
  <img src="../images/agent deployment/16-two-agents-active-dashboard.png" width="900">
</p>

Additional validation confirmed that Sysmon Operational events from the Windows 11 workstation were successfully ingested and searchable within the Threat Hunting interface.

<p align="center">
  <img src="../images/agent deployment/17-win11-sysmon-events-ingested.png" width="900">
</p>

The completed validation demonstrated:

- Successful registration of both Windows endpoints.
- Active communication with the Wazuh Manager.
- Windows Security event collection.
- Sysmon Operational event collection.
- End-to-end telemetry ingestion into the SIEM platform.

---

# Deployment Summary

This phase completed the integration of Windows endpoints into the Wazuh SIEM environment.

Both the Domain Controller and Windows 11 workstation were successfully onboarded, authenticated with the Wazuh Manager, and continuously transmitted Windows Security and Sysmon telemetry to the platform. Endpoint visibility was verified through the Wazuh Dashboard and Threat Hunting interface, confirming that the logging pipeline was operating correctly across the environment.

With endpoint onboarding complete, the SIEM platform is fully prepared for the next phase of the project, which focuses on attack simulation, detection engineering, alert analysis, and threat hunting.