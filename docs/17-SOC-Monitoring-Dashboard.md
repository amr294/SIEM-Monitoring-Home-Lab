# 17 - SOC Monitoring Dashboard

## 1. Objective

The objective of this stage was to extend the existing Wazuh deployment
from centralized event collection into a practical security-monitoring
and investigation workflow.

The dashboard was designed around the telemetry already available from:

-   `DC01` --- Windows Server 2022 / Active Directory Domain Controller
-   `WIN11-CLIENT` --- Windows 11 endpoint
-   `amr-server` --- Wazuh SIEM host
-   `FortiGate-VM` --- network-security gateway

The implementation focused on three areas:

1.  **Security monitoring** --- provide a consolidated view of Wazuh
    alert activity.
2.  **Event investigation** --- create reusable OpenSearch saved
    searches for endpoint and network investigation.
3.  **Telemetry validation** --- confirm that Windows and FortiGate
    events are available at the event level and can be investigated
    through the SIEM.

The resulting workflow is:

``` text
Windows / FortiGate telemetry
          ↓
      Wazuh Manager
          ↓
     wazuh-alerts-*
          ↓
 OpenSearch Dashboard
          ↓
 Monitoring / Investigation
```

This stage builds on the existing FortiGate → Wazuh integration
documented in [16 - FortiGate Wazuh
Integration](16-FortiGate-Wazuh-Integration.md).

------------------------------------------------------------------------

## 2. Monitoring Data Source

The dashboard and saved searches use:

``` text
wazuh-alerts-*
```

The captured visualizations use the Wazuh alert fields already produced
by the existing monitoring pipeline.

The main fields used in this stage are:

``` text
timestamp
agent.name
rule.level
rule.description
rule.id
decoder.name
data.type
```

Windows-specific investigation uses fields including:

``` text
data.win.system.eventID
data.win.eventdata.logonType
data.win.eventdata.ipAddress
data.win.eventdata.ipPort
data.win.eventdata.targetUserName
data.win.eventdata.processName
data.win.eventdata.commandLine
```

FortiGate investigation uses:

``` text
data.srcip
data.srcport
data.dstip
data.dstport
data.service
data.action
data.policyid
data.srccountry
data.dstcountry
```

------------------------------------------------------------------------

## 3. SOC Dashboard

The main dashboard consolidates nine visualizations into a single
monitoring view.

![SOC Security Monitoring
Dashboard](../images/17-SOC-Monitoring-Dashboard/01_SOC_Security_Monitoring_Dashboard_9_Panels.png)

**Figure 1 --- SOC Security Monitoring Dashboard containing endpoint,
authentication, telemetry, and FortiGate monitoring panels.**

The dashboard contains:

  -----------------------------------------------------------------------
  Panel                               Purpose
  ----------------------------------- -----------------------------------
  SOC - Alerts Over Time              Monitor alert volume and identify
                                      activity spikes

  SOC - Alerts by Agent               Identify the monitored systems
                                      generating the highest alert volume

  SOC - Alerts by Severity            Review the distribution of Wazuh
                                      rule levels

  SOC - Telemetry Sources             Identify the decoders generating
                                      the observed alerts

  SOC - Windows Authentication        Monitor successful Windows logon
                                      activity

  SOC - Windows Failed Authentication Identify source IPs associated with
  Sources                             failed Windows authentication

  SOC - FortiGate Traffic Alerts Over Monitor FortiGate traffic activity
  Time                                over time

  SOC - FortiGate Top Traffic Sources Identify the most active FortiGate
                                      source IPs

  SOC - FortiGate Traffic Actions     Review observed firewall actions
  -----------------------------------------------------------------------

The dashboard was configured with a **Last 1 month** time range in the
captured evidence.

The dashboard is intended as the first layer of the investigation
process. Individual events are investigated through the saved searches
described later in this document.

------------------------------------------------------------------------

# 4. Alert Monitoring

## 4.1 Alerts Over Time

![Alerts Over
Time](../images/17-SOC-Monitoring-Dashboard/02_SOC_Alerts_Over_Time_Configuration.png)

**Figure 2 --- Alerts Over Time visualization configuration.**

### Configuration

  Field              Value
  ------------------ ------------------
  Data source        `wazuh-alerts-*`
  Metric             Count
  X-axis             Date Histogram
  Field              `timestamp`
  Minimum interval   Auto
  Custom label       Count

The visualization uses a date histogram over `timestamp` to display the
number of Wazuh alert documents generated during each time bucket.

The captured results show multiple activity spikes, including a
significantly larger increase toward the end of the displayed period.

This provides the initial indication that a period requires further
investigation.

The workflow is:

``` text
Alert spike
    ↓
Identify time window
    ↓
Identify affected agent
    ↓
Review rule level
    ↓
Identify telemetry source
    ↓
Inspect individual events
```

The visualization therefore acts as an alert-volume monitoring layer
rather than a standalone incident detector.

------------------------------------------------------------------------

## 4.2 Alerts by Agent

![Alerts by
Agent](../images/17-SOC-Monitoring-Dashboard/03_SOC_Alerts_by_Agent_Configuration.png)

**Figure 3 --- Alerts by Agent visualization configuration.**

### Configuration

  Field          Value
  -------------- ------------------
  Data source    `wazuh-alerts-*`
  Metric         Count
  Metric label   `Alert Count`
  Aggregation    Terms
  Field          `agent.name`
  Order          Alert Count
  Direction      Descending
  Size           5

The captured results contain the following monitored agents:

``` text
amr-server
WIN11-CLIENT
DC01
```

`amr-server` represents the highest alert volume in the captured
dashboard, followed by `WIN11-CLIENT` and `DC01`.

The result does not by itself indicate that `amr-server` is the most
compromised system.

Alert volume can be affected by:

-   Number of monitored services
-   Log volume
-   File integrity monitoring
-   Authentication activity
-   System activity
-   Detection configuration

The panel is therefore used to identify where the analyst should begin
investigating rather than to determine compromise by itself.

------------------------------------------------------------------------

## 4.3 Alerts by Severity

![Alerts by
Severity](../images/17-SOC-Monitoring-Dashboard/04_SOC_Alerts_by_Severity_Configuration.png)

**Figure 4 --- Alerts by Severity visualization configuration.**

### Configuration

  Field          Value
  -------------- ------------------
  Data source    `wazuh-alerts-*`
  Metric         Count
  Metric label   `Alert Count`
  Aggregation    Terms
  Field          `rule.level`
  Order          Alert Count
  Direction      Descending
  Size           10

The captured dataset contains Wazuh rule levels ranging from
lower-severity events through higher-level alerts.

The most prominent levels are:

``` text
3
7
5
6
4
9
10
15
8
12
```

Level `3` represents the largest portion of the captured alert
population.

The visualization is used to distinguish between **alert volume** and
**alert severity**.

A high event count at a lower rule level should not automatically
receive the same priority as a smaller number of high-severity alerts.

------------------------------------------------------------------------

## 4.4 Telemetry Sources

![Telemetry
Sources](../images/17-SOC-Monitoring-Dashboard/05_SOC_Telemetry_Sources_Configuration.png)

**Figure 5 --- Telemetry Sources visualization configuration.**

### Configuration

  Field          Value
  -------------- --------------------
  Data source    `wazuh-alerts-*`
  Metric         Count
  Metric label   `Alert Count`
  Aggregation    Terms
  Field          `decoder.name`
  Order          Alert Count
  Direction      Descending
  Size           10
  Custom label   `Telemetry Source`

The captured results include:

``` text
windows_eventchannel
rootcheck
dpkg-decoder
pam
syscheck_registry_value_added
ossec
sudo
json
syscheck_registry_value_modified
kernel
```

`windows_eventchannel` is the dominant decoder in the captured dataset.

This provides visibility into which telemetry sources are contributing
most of the observed alerts.

The result can also be used to identify unexpected changes in the
telemetry profile or unusually noisy event sources.

------------------------------------------------------------------------

# 5. Windows Authentication Monitoring

## 5.1 Successful Authentication

![Windows
Authentication](../images/17-SOC-Monitoring-Dashboard/06_SOC_Windows_Authentication_Configuration.png)

**Figure 6 --- Windows Authentication visualization configuration.**

The visualization uses:

``` text
data.win.system.eventID:4624
```

### Configuration

  Field          Value
  -------------- --------------------------------
  Data source    `wazuh-alerts-*`
  Metric         Count
  Metric label   `Authentication Events`
  Aggregation    Terms
  Field          `data.win.eventdata.logonType`
  Order          Authentication Events
  Direction      Descending
  Size           10
  Custom label   `Windows Logon Type`

The captured results are dominated by:

``` text
Logon Type 3
```

Windows Event ID `4624` represents a successful logon.

The visualization groups these successful logons by Windows logon type
to provide a basic authentication profile for the monitored environment.

The result is not interpreted as malicious activity by itself.

Authentication analysis requires additional context such as:

``` text
User
Source IP
Logon Type
Timestamp
Authentication Package
Destination Host
Related Failed Logons
```

------------------------------------------------------------------------

## 5.2 Failed Authentication Sources

![Windows Failed Authentication
Sources](../images/17-SOC-Monitoring-Dashboard/07_SOC_Windows_Failed_Authentication_Sources_Configuration.png)

**Figure 7 --- Windows Failed Authentication Sources visualization
configuration.**

The visualization uses:

``` text
data.win.system.eventID:4625
```

### Configuration

  Field          Value
  -------------- --------------------------------
  Data source    `wazuh-alerts-*`
  Metric         Count
  Metric label   `Failed Authentication Events`
  Aggregation    Terms
  Field          `data.win.eventdata.ipAddress`
  Order          Failed Authentication Events
  Direction      Descending
  Size           10
  Custom label   `Source IP`

The captured result is dominated by:

``` text
127.0.0.1
```

`127.0.0.1` is the local loopback address.

The result therefore cannot be treated as evidence of an external
brute-force attack.

The correct investigation process is to inspect:

-   Target username
-   Authentication package
-   Failure status
-   Failure substatus
-   Process context where available
-   Related successful logons
-   Event timestamps

This demonstrates why an individual field must be interpreted within the
complete event context.

------------------------------------------------------------------------

# 6. FortiGate Traffic Monitoring

The FortiGate dashboard panels use the following filter:

``` text
decoder.name:fortigate-firewall-v5 AND data.type:traffic
```

This restricts the visualization to FortiGate traffic events processed
by Wazuh.

------------------------------------------------------------------------

## 6.1 FortiGate Traffic Alerts Over Time

![FortiGate Traffic Alerts Over
Time](../images/17-SOC-Monitoring-Dashboard/08_SOC_FortiGate_Traffic_Alerts_Over_Time_Configuration.png)

**Figure 8 --- FortiGate Traffic Alerts Over Time visualization
configuration.**

### Configuration

  Field              Value
  ------------------ ------------------------------------------------------------
  Data source        `wazuh-alerts-*`
  Filter             `decoder.name:fortigate-firewall-v5 AND data.type:traffic`
  Metric             Count
  Metric label       `Traffic Alerts`
  X-axis             Date Histogram
  Field              `timestamp`
  Minimum interval   Auto

The captured results show FortiGate traffic activity concentrated toward
the end of the displayed period.

This visualization provides a temporal view of firewall traffic events
and can be used to identify periods requiring network-level
investigation.

------------------------------------------------------------------------

## 6.2 FortiGate Top Traffic Sources

![FortiGate Top Traffic
Sources](../images/17-SOC-Monitoring-Dashboard/09_SOC_FortiGate_Top_Traffic_Sources_Configuration.png)

**Figure 9 --- FortiGate Top Traffic Sources visualization
configuration.**

### Configuration

  Field          Value
  -------------- ------------------------------------------------------------
  Data source    `wazuh-alerts-*`
  Filter         `decoder.name:fortigate-firewall-v5 AND data.type:traffic`
  Metric         Count
  Metric label   `Traffic Alerts`
  Aggregation    Terms
  Field          `data.srcip`
  Order          Traffic Alerts
  Direction      Descending
  Size           10
  Custom label   `Source IP`

The captured results contain:

``` text
10.10.10.102
10.10.10.101
10.10.10.100
```

`10.10.10.102` is the dominant source in the captured visualization.

The panel provides a starting point for identifying high-volume sources
and correlating network activity with endpoint telemetry.

High traffic volume alone does not establish malicious activity.

------------------------------------------------------------------------

## 6.3 FortiGate Traffic Actions

![FortiGate Traffic
Actions](../images/17-SOC-Monitoring-Dashboard/10_SOC_FortiGate_Traffic_Actions_Configuration.png)

**Figure 10 --- FortiGate Traffic Actions visualization configuration.**

### Configuration

  Field          Value
  -------------- ------------------------------------------------------------
  Data source    `wazuh-alerts-*`
  Filter         `decoder.name:fortigate-firewall-v5 AND data.type:traffic`
  Metric         Count
  Metric label   `Traffic Events`
  Aggregation    Terms
  Field          `data.action`
  Order          Traffic Events
  Direction      Descending
  Size           2
  Custom label   `Firewall Action`

The captured visualization contains:

  Action       Count
  ---------- -------
  `accept`        24
  `close`         10

The action field should be investigated together with:

``` text
Source IP
Destination IP
Destination Port
Service
Policy ID
Source Country
Destination Country
```

This provides the context required to determine what traffic the
firewall is actually handling.

------------------------------------------------------------------------

# 7. Saved Searches

## 7.1 Saved Search Inventory

![Saved
Searches](../images/17-SOC-Monitoring-Dashboard/11_SOC_Saved_Searches.png)

**Figure 11 --- OpenSearch saved searches used for event-level
investigation.**

The saved-search interface contains:

``` text
Domain Controller Events
SOC - FortiGate Traffic Events
Windows Client Events
```

These searches provide the second layer of the monitoring workflow.

The dashboard provides aggregated visibility:

``` text
What is happening?
```

The saved searches provide event-level visibility:

``` text
Which events produced the activity?
```

This distinction is important because aggregate visualizations cannot
provide all of the fields required for investigation.

------------------------------------------------------------------------

# 8. Domain Controller Event Investigation

![Domain Controller
Events](../images/17-SOC-Monitoring-Dashboard/12_SOC_Domain_Controller_Events.png)

**Figure 12 --- Domain Controller event investigation using OpenSearch
Discover.**

The saved search uses:

``` text
agent.name:"DC01" AND decoder.name:"windows_eventchannel"
```

The captured search uses a:

``` text
Last 24 hours
```

time range and returned:

``` text
783 hits
```

The event table contains fields including:

``` text
agent.name
data.win.system.eventID
rule.level
rule.description
data.win.eventdata.targetUserName
data.win.eventdata.ipAddress
data.win.eventdata.ipPort
data.win.eventdata.authenticationPackageName
data.win.system.channel
```

The visible events include:

    Event ID Description
  ---------- -------------------------------------------
        4647 Windows User Logoff
        6000 SessionEnv notification event unavailable
        1074 System shutdown initiated
        4634 Windows User Logoff
        4624 Windows Logon Success

The saved search provides direct visibility into Domain Controller
activity and allows the analyst to pivot from dashboard-level activity
into the underlying Windows events.

------------------------------------------------------------------------

# 9. FortiGate Event Investigation

![FortiGate Traffic
Events](../images/17-SOC-Monitoring-Dashboard/13_SOC_FortiGate_Traffic_Events.png)

**Figure 13 --- FortiGate traffic event investigation using OpenSearch
Discover.**

The saved search uses:

``` text
decoder.name:"fortigate-firewall-v5"
```

The captured search uses a:

``` text
Last 1 month
```

time range and returned:

``` text
71 hits
```

The event table exposes:

``` text
data.srcip
data.srcport
data.dstip
data.dstport
data.service
data.action
data.policyid
data.srccountry
data.dstcountry
```

One visible event contains:

``` text
Source IP        10.10.10.102
Source Port      61138
Destination      95.101.35.96
Destination Port 443
Service          HTTPS
Action           accept
Policy ID        1
Destination      Belgium
```

Other captured events include DNS and HTTP traffic as well as FortiGate
login/logout events.

The event-level search therefore provides the information required to
investigate the traffic source, destination, service, action, and policy
rather than relying only on the aggregated dashboard.

------------------------------------------------------------------------

# 10. Windows Client Event Investigation

![Windows Client
Events](../images/17-SOC-Monitoring-Dashboard/14_SOC_Windows_Client_Events.png)

**Figure 14 --- Windows 11 endpoint event investigation using OpenSearch
Discover.**

The saved search uses:

``` text
agent.name:"WIN11-CLIENT" AND decoder.name:"windows_eventchannel"
```

The captured search also contains:

``` text
manager.name:amr-server
```

The search uses a:

``` text
Last 1 month
```

time range and returned:

``` text
1,918 hits
```

The event table contains:

``` text
agent.name
rule.description
data.win.system.eventID
rule.level
data.win.eventdata.targetUserName
```

The captured events include:

    Event ID Description
  ---------- -------------------------------------------
        4688 A process was created
        4647 Windows User Logoff
        6000 SessionEnv notification event unavailable
        1074 System shutdown initiated

Event ID `4688` provides process creation telemetry and is particularly
useful when investigating process execution and command-line activity.

The saved search therefore provides an endpoint investigation pivot from
the main SOC dashboard.

------------------------------------------------------------------------

# 11. Dashboard Investigation Methodology

The dashboard was designed around the following investigation sequence:

``` text
1. Identify abnormal activity
          ↓
2. Identify the affected asset
          ↓
3. Review alert severity
          ↓
4. Identify the telemetry source
          ↓
5. Filter the relevant event stream
          ↓
6. Inspect individual events
          ↓
7. Correlate endpoint and network activity
          ↓
8. Determine the event context
```

The methodology intentionally separates **monitoring** from
**investigation**.

The dashboard provides the initial visibility required to identify
activity.

OpenSearch Discover and the saved searches provide the detailed event
context required to investigate that activity.

------------------------------------------------------------------------

# 12. Cross-Telemetry Correlation

The current environment now provides both Windows endpoint telemetry and
FortiGate network telemetry through Wazuh.

The intended investigation path is:

``` text
Windows Authentication
        ↓
User / Source IP / Timestamp
        ↓
Windows Endpoint Events
        ↓
Process / Logon / System Activity
        ↓
FortiGate Traffic
        ↓
Source / Destination / Port / Service / Action
```

This creates the foundation for correlating:

``` text
Identity
    +
Endpoint
    +
Network
```

The current dashboard provides visibility into each of these layers.

Full automated correlation between the layers is not represented as
implemented in this stage.

------------------------------------------------------------------------

# 13. Validation Summary

  Validation                                                       Status
  --------------------------------------------------------- ---------------------
  Wazuh alert data available                                         ✅
  Main SOC dashboard created                                         ✅
  Alert timeline visualization created                               ✅
  Agent distribution visualization created                           ✅
  Severity distribution visualization created                        ✅
  Telemetry-source visualization created                             ✅
  Windows successful authentication visualization created            ✅
  Windows failed authentication visualization created                ✅
  FortiGate traffic timeline created                                 ✅
  FortiGate source visualization created                             ✅
  FortiGate action visualization created                             ✅
  Saved searches created                                             ✅
  Domain Controller event investigation validated                    ✅
  FortiGate event investigation validated                            ✅
  Windows client event investigation validated                       ✅
  Endpoint/network correlation foundation established                ✅
  Full automated cross-source correlation                    Not yet implemented

------------------------------------------------------------------------

# 14. Current Results

The completed dashboard provides a consolidated monitoring layer over
the existing Wazuh deployment.

The captured environment demonstrates:

-   Multiple monitored Wazuh agents.
-   Windows Event Channel telemetry.
-   Linux host telemetry.
-   Wazuh rule-level alert distribution.
-   Windows authentication visibility.
-   Windows failed-authentication source visibility.
-   FortiGate traffic telemetry.
-   FortiGate source-IP visibility.
-   FortiGate firewall-action visibility.
-   Event-level investigation through OpenSearch saved searches.

The captured dashboard identifies:

``` text
Highest displayed agent alert volume:
amr-server

Dominant telemetry source:
windows_eventchannel

Dominant Windows authentication logon type:
3

Dominant captured failed-authentication source:
127.0.0.1

Dominant FortiGate traffic source:
10.10.10.102
```

These values describe the captured dataset and should not be interpreted
as permanent baseline values for the lab.

------------------------------------------------------------------------

# 15. Investigation Considerations

The captured results also demonstrate several cases where additional
context is required.

### Failed authentication source

The `127.0.0.1` result must not automatically be classified as an
external authentication attack because it represents the local loopback
address.

### High alert volume

A high alert count on `amr-server` does not automatically establish
compromise. The underlying alert types and rule levels must be
investigated.

### High traffic source

The high traffic volume from `10.10.10.102` does not automatically
indicate malicious network behavior.

### Low-level alerts

A large number of level `3` alerts can represent normal or informational
activity and should not be treated as equivalent to high-severity
detections.

These examples reinforce the purpose of the dashboard:

``` text
Detection
   ↓
Prioritization
   ↓
Investigation
   ↓
Context
   ↓
Disposition
```

The dashboard provides the first stages of this workflow; the analyst
remains responsible for determining the final event disposition.

------------------------------------------------------------------------

# 16. Lessons Learned

-   A SIEM dashboard is most useful when it provides a direct path from
    aggregated activity to individual events.
-   Alert volume alone is not sufficient to determine security impact.
-   Rule severity should be evaluated together with event context.
-   Decoder distribution provides useful visibility into the sources
    contributing to SIEM activity.
-   Windows authentication events provide an important starting point
    for identity-related investigations.
-   Source IP analysis requires awareness of local addresses and network
    context.
-   FortiGate traffic telemetry adds network-level context to the
    existing Windows endpoint monitoring.
-   Saved searches reduce the time required to pivot from dashboard
    activity into raw events.
-   Endpoint and network telemetry provide stronger investigation
    capability when analyzed together.
-   The current implementation establishes the monitoring foundation
    required for future detection engineering and threat-hunting work.

------------------------------------------------------------------------

# 17. Current Scope and Limitations

This stage focuses on dashboard monitoring and event-level
investigation.

The current implementation demonstrates:

-   Wazuh alert visualization.
-   Windows event monitoring.
-   Authentication monitoring.
-   FortiGate traffic monitoring.
-   OpenSearch saved searches.
-   Event-level investigation.

The current evidence does not establish a complete automated correlation
engine between Windows and FortiGate events.

The following work remains for later stages:

``` text
Network / Endpoint Correlation
          ↓
Custom Detection Engineering
          ↓
Threat Hunting
          ↓
Incident Investigation Scenarios
          ↓
Response Workflow
```

The dashboard should therefore be considered the monitoring and
investigation foundation rather than the final state of the SOC
implementation.

------------------------------------------------------------------------

# 18. Next Stage

The next stage can build directly on the telemetry and investigation
workflow established here.

Planned development includes:

-   Correlating Windows authentication with FortiGate source IPs.
-   Investigating successful logons following repeated failures.
-   Monitoring Windows process creation.
-   Developing custom Wazuh detection rules.
-   Creating FortiGate denied-traffic use cases.
-   Investigating VPN and administrative events.
-   Building threat-hunting searches.
-   Developing controlled incident scenarios.
-   Mapping detections to MITRE ATT&CK.
-   Documenting investigation and response procedures.

The objective is to move from:

``` text
Security Monitoring
```

toward:

``` text
Detection
    ↓
Investigation
    ↓
Correlation
    ↓
Threat Hunting
    ↓
Incident Response
```

------------------------------------------------------------------------

## Related Documentation

-   [00 - Current Lab Architecture](00-Current-Lab-Architecture.md)
-   [10 - Wazuh SIEM Deployment](10-Wazuh-SIEM-Deployment.md)
-   [11 - Wazuh Agent Deployment](11-Wazuh-Agent-Deployment.md)
-   [12 - Attack Simulation](12-Attack-Simulation.md)
-   [13 - FortiGate Network Edge](13-fortigate-network-edge.md)
-   [14 - FortiGate Remote Access
    VPN](14-FortiGate-Remote-Access-VPN.md)
-   [15 - Wazuh Dashboard Port
    Migration](15-Wazuh-Dashboard-Port-Migration.md)
-   [16 - FortiGate Wazuh
    Integration](16-FortiGate-Wazuh-Integration.md)

------------------------------------------------------------------------

## Evidence

The dashboard evidence for this stage is maintained as a separate
evidence package containing the dashboard, visualization configurations,
saved-search inventory, and event-level investigation screenshots.

The evidence package corresponds to Figures 1--14 in this document.
