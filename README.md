
# Microsoft Sentinel Honeypot | SIEM & Threat Detection Lab

## Project Overview

This project demonstrates the deployment of a Windows honeypot in Microsoft Azure to monitor suspicious authentication activity, collect Windows Security Events, and visualize failed login attempts using Microsoft Sentinel.

The lab integrates Azure Virtual Machines, Azure Monitor Agent (AMA), Log Analytics, Microsoft Sentinel, Kusto Query Language (KQL), and GeoIP enrichment to create a centralized security monitoring environment.

The primary objective is to investigate failed authentication attempts (Windows Event ID 4625) and create an interactive world map displaying the approximate geographic origins of the source IP addresses.

> **Security Notice:** This project was conducted in an isolated, disposable lab environment. The permissive firewall and network configurations used for the honeypot are not recommended for production environments.

---

## Table of Contents

1. [Project Objectives](#project-objectives)
2. [Technologies Used](#technologies-used)
3. [Deploy Azure Virtual Machine](#step-1-deploy-azure-virtual-machine)
4. [Configure Inbound Security Rules](#step-2-configure-inbound-security-rules)
5. [Configure Windows Firewall](#step-3-configure-windows-firewall)
6. [Generate and Analyze Failed Logins](#step-4-generate-and-analyze-failed-logins)
7. [Create Log Analytics Workspace](#step-5-create-log-analytics-workspace)
8. [Configure Microsoft Sentinel](#step-6-configure-microsoft-sentinel)
9. [Install Windows Security Events Connector](#step-7-install-windows-security-events-connector)
10. [Create Data Collection Rule](#step-8-create-data-collection-rule)
11. [Verify Azure Monitor Agent](#step-9-verify-azure-monitor-agent)
12. [Analyze Security Events Using KQL](#step-10-analyze-security-events-using-kql)
13. [Configure GeoIP Watchlist](#step-11-configure-geoip-watchlist)
14. [Create Honeypot Attack Map](#step-12-create-honeypot-attack-map)
15. [Security Findings](#security-findings)
16. [Challenges and Troubleshooting](#challenges-and-troubleshooting)
17. [Lessons Learned](#lessons-learned)
18. [Lab Cleanup and Security](#lab-cleanup-and-security)
19. [Conclusion](#conclusion)

---

## Project Objectives

- Deploy a Windows virtual machine in Azure as a honeypot.
- Configure an isolated lab environment to observe unsolicited network traffic.
- Generate and investigate Windows authentication events.
- Collect and centralize VM security logs using Azure Log Analytics.
- Integrate Log Analytics with Microsoft Sentinel.
- Configure Azure Monitor Agent and Data Collection Rules.
- Use KQL to analyze failed login attempts and source IP addresses.
- Enrich security logs using a GeoIP watchlist.
- Develop an interactive attack map showing approximate geographic locations.
- Gain practical experience with SIEM operations, log analysis, and threat monitoring.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Azure | Cloud infrastructure |
| Windows Virtual Machine | Honeypot system |
| Azure Network Security Group | Network traffic configuration |
| Windows Event Viewer | Local security log analysis |
| Azure Log Analytics | Centralized log repository |
| Microsoft Sentinel | SIEM and threat monitoring |
| Azure Monitor Agent (AMA) | Security event collection |
| Data Collection Rule (DCR) | Defines collected security events |
| Kusto Query Language (KQL) | Log analysis and filtering |
| Microsoft Sentinel Watchlists | GeoIP reference data |
| Microsoft Sentinel Workbooks | Geographic attack visualization |

---

# Implementation

## Step 1: Deploy Azure Virtual Machine

The first step was creating a Windows virtual machine in Microsoft Azure.

This VM acts as the honeypot, allowing me to observe failed authentication attempts and collect security telemetry for analysis.

**VM Configuration:**

- Cloud Platform: Microsoft Azure
- Operating System: Windows
- VM Size: Standard D2s v3
- Purpose: Honeypot and security event monitoring

For detailed instructions on creating an Azure VM, refer to my existing GitHub documentation:

**[Azure VM Deployment Guide](INSERT_YOUR_VM_GITHUB_LINK_HERE)**

### Screenshot: Azure VM

![Azure VM](images/01-azure-vm.png)

[Back to Top](#table-of-contents)

---

## Step 2: Configure Inbound Security Rules

After creating the VM, I configured an inbound security rule using Azure Network Security Groups (NSGs).

The rule was intentionally configured to permit all inbound traffic within the isolated lab environment.

This configuration allowed the honeypot to receive unsolicited connection attempts.

**Configuration:**

- Source: Any
- Source Port Ranges: *
- Destination: Any
- Destination Port Ranges: *
- Protocol: Any
- Action: Allow

**Security Note:** This configuration significantly increases exposure and should only be used in a disposable, isolated honeypot environment without sensitive information. Production environments should restrict traffic to authorized sources and necessary ports.

### Screenshot: Inbound Security Rule

![Inbound Security Rule](images/02-inbound-security-rule.png)

[Back to Top](#table-of-contents)

---

## Step 3: Configure Windows Firewall

After configuring Azure networking, I connected to the Windows VM.

Connection methods included:

- Azure Bastion
- Remote Desktop Connection (RDP)

Within the lab environment, I disabled the internal Windows Defender Firewall to allow the honeypot to receive additional incoming network traffic.

I also tested network connectivity from my host computer using the VM's public IP address.

```powershell
ping <VM-PUBLIC-IP>
```

A successful ping response can confirm ICMP reachability, although it does not independently verify RDP connectivity.

### Screenshot: Windows Firewall

![Windows Firewall](images/03-windows-firewall.png)

[Back to Top](#table-of-contents)

---

## Step 4: Generate and Analyze Failed Logins

To verify Windows Security Event logging, I deliberately attempted to authenticate to the VM using incorrect credentials approximately four to five times.

After generating the failed login attempts, I logged into the VM using the correct credentials.

I opened Windows Event Viewer and navigated to:

**Event Viewer → Windows Logs → Security**

Event Viewer records security-related activities occurring on the Windows operating system.

### Important Windows Event IDs

| Event ID | Description |
|---|---|
| 4625 | Failed login attempt |
| 4624 | Successful login |

**Event ID 4625** was particularly important because it records failed authentication attempts.

Selecting individual security events allows examination of available details, including:

- Time of the authentication attempt
- Account information
- Logon type
- Source IP address, when available
- Authentication failure details

These logs provide evidence that can help security analysts identify suspicious authentication patterns.

### Screenshot: Failed Login Attempts

![Failed Login Attempts](images/04-failed-login-events.png)

[Back to Top](#table-of-contents)

---

## Step 5: Create Log Analytics Workspace

The next step was creating a centralized log repository using Azure Log Analytics.

Log Analytics allows administrators and security analysts to collect, store, query, and investigate security events from connected Azure resources.

### Configuration Steps

1. Open the Azure Portal.
2. Search for **Log Analytics workspaces**.
3. Select **Create**.
4. Choose the subscription and resource group.
5. Enter the workspace name.
6. Select an Azure region.
7. Review and create the workspace.

This workspace would later receive security events from the honeypot VM.

### Screenshot: Log Analytics Workspace

![Log Analytics Workspace](images/05-log-analytics-workspace.png)

[Back to Top](#table-of-contents)

---

## Step 6: Configure Microsoft Sentinel

After deploying the Log Analytics workspace, I configured Microsoft Sentinel.

Microsoft Sentinel is a cloud-native Security Information and Event Management (SIEM) solution used to collect, analyze, investigate, and respond to security events.

### Configuration Steps

1. Search for **Microsoft Sentinel** in Azure.
2. Select **Create**.
3. Choose the previously created Log Analytics workspace.
4. Add Microsoft Sentinel to the workspace.

This integration enables security monitoring and analysis using the data stored in Log Analytics.

### Screenshot: Microsoft Sentinel

![Microsoft Sentinel](images/06-microsoft-sentinel.png)

[Back to Top](#table-of-contents)

---

## Step 7: Install Windows Security Events Connector

To forward Windows Security Events from the honeypot to Log Analytics, I configured the Windows Security Events solution using Azure Monitor Agent.

### Configuration Steps

1. Open Microsoft Sentinel.
2. Navigate to **Content Management → Content Hub**.
3. Search for **Windows Security Events**.
4. Select **Install**.
5. After installation, select **Manage**.
6. Open **Windows Security Events via AMA**.
7. Click **Open Connector Page**.

This connector provides the configuration required to collect Windows Security Events through the Azure Monitor Agent.

### Screenshot: Windows Security Events Connector

![Windows Security Events Connector](images/07-windows-security-connector.png)

[Back to Top](#table-of-contents)

---

## Step 8: Create Data Collection Rule

After installing the connector, I created a Data Collection Rule (DCR).

A Data Collection Rule defines which data should be collected from the connected VM and where that data should be sent.

### Configuration Steps

1. Select **Create Data Collection Rule**.
2. Enter a name for the rule.
3. Navigate to the **Resources** tab.
4. Select the previously created honeypot VM.
5. Navigate to the **Collect** tab.
6. Select **All Security Events**.
7. Review the configuration.
8. Click **Create**.

**Collection Setting:**

```text
Security Events: All Security Events
Target Resource: Windows Honeypot VM
Destination: Log Analytics Workspace
```

Selecting all security events allows collection of broader Windows security telemetry, including successful and failed authentication events.

### Screenshot: Data Collection Rule

![Data Collection Rule](images/08-data-collection-rule.png)

[Back to Top](#table-of-contents)

---

## Step 9: Verify Azure Monitor Agent

After creating the Data Collection Rule, I verified that the Azure Monitor Agent extension was installed on the VM.

### Verification Steps

1. Open the Azure Portal.
2. Navigate to **Virtual Machines**.
3. Select the honeypot VM.
4. Open **Settings → Extensions + Applications**.
5. Verify the presence of:

```text
AzureMonitorWindowsAgent
```

The agent enables Windows event collection based on the configured Data Collection Rule.

The presence of the extension confirms installation, while successful query results in Log Analytics confirm that security event data is being received.

### Screenshot: Azure Monitor Agent

![Azure Monitor Agent](images/09-azure-monitor-agent.png)

[Back to Top](#table-of-contents)

---

## Step 10: Analyze Security Events Using KQL

Once the Azure Monitor Agent and Data Collection Rule were configured, security events began appearing in the Log Analytics workspace.

I used Kusto Query Language (KQL) to investigate authentication activity.

KQL allows security analysts to search, filter, aggregate, and analyze large volumes of log data.

### Query 1: Identify Failed Login Attempts

```kql
SecurityEvent
| where EventID == 4625
| project TimeGenerated, Computer, Account, IpAddress
| order by TimeGenerated desc
```

**Explanation:**

- `SecurityEvent` retrieves Windows Security Event logs.
- `EventID == 4625` filters failed login attempts.
- `project` selects the relevant columns.
- `order by` displays the newest events first.

### Query 2: Count Failed Login Attempts by IP Address

```kql
SecurityEvent
| where EventID == 4625
| where isnotempty(IpAddress)
| summarize FailedAttempts = count() by IpAddress
| order by FailedAttempts desc
```

This query helps identify source IP addresses associated with repeated authentication failures.

### Query 3: Identify Successful Logins

```kql
SecurityEvent
| where EventID == 4624
| project TimeGenerated, Computer, Account, IpAddress, LogonType
| order by TimeGenerated desc
```

This query retrieves successful authentication events.

For successful Remote Desktop authentication events, Logon Type 10 is particularly relevant.

### Screenshot: KQL Analysis

![KQL Analysis](images/10-kql-failed-logins.png)

[Back to Top](#table-of-contents)

---

## Step 11: Configure GeoIP Watchlist

Although Windows Security Event logs may contain the source IP address of an authentication attempt, they do not typically contain geographic coordinates.

To determine approximate geographic locations, I imported a GeoIP CSV file into Microsoft Sentinel Watchlists.

The CSV file provides reference information for associating IP address ranges with geographic locations.

### Configuration Steps

1. Open Microsoft Sentinel.
2. Navigate to **Configuration → Watchlists**.
3. Open the Microsoft Defender portal.
4. Select **New Watchlist**.
5. Enter the watchlist name and alias.
6. Upload the GeoIP CSV file.
7. Set the search key to `network`.
8. Review and create the watchlist.

### Watchlist Configuration

| Setting | Value |
|---|---|
| Alias | geoip |
| Source Type | CSV |
| Search Key | network |
| Purpose | Geographic IP enrichment |
| Reference Records | Approximately 55,000 |

**Important:** The approximately 55,000 watchlist entries represent geographic reference records, not actual login attempts or detected attacks.

These entries are used to match source IP addresses from security events to geographic information.

### Screenshot: GeoIP Watchlist

![GeoIP Watchlist](images/11-geoip-watchlist.png)

[Back to Top](#table-of-contents)

---

## Step 12: Create Honeypot Attack Map

After importing the GeoIP watchlist, I created a Microsoft Sentinel Workbook to visualize failed login attempts geographically.

The workbook combines Windows Security Events with the GeoIP reference information to display approximate source locations on an interactive map.

### Configuration Steps

1. Open Microsoft Sentinel.
2. Navigate to **Threat Management → Workbooks**.
3. Open the Microsoft Defender portal.
4. Select **Add Workbook**.
5. Click **Edit**.
6. Remove unnecessary prepopulated workbook content.
7. Click **Add data source + visualization**.
8. Select the Log Analytics data source.
9. Enter the KQL query.
10. Run the query.
11. Select the **Map** visualization.
12. Configure latitude, longitude, and failure count.
13. Save the workbook.

### KQL Query: Geographic Attack Map

```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent;
WindowsEvents
| where EventID == 4625
| order by TimeGenerated desc
| evaluate ipv4_lookup(
    GeoIPDB_FULL,
    IpAddress,
    network
)
| summarize FailureCount = count()
    by IpAddress, latitude, longitude,
       cityname, countryname
| project
    FailureCount,
    AttackerIp = IpAddress,
    latitude,
    longitude,
    city = cityname,
    country = countryname,
    friendly_location =
        strcat(cityname, " (", countryname, ")")
```

### Query Explanation

**1. Retrieve GeoIP Reference Data**

```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
```

Retrieves the GeoIP watchlist previously uploaded to Microsoft Sentinel.

**2. Retrieve Windows Security Events**

```kql
let WindowsEvents = SecurityEvent;
```

References the collected Windows security logs.

**3. Filter Failed Authentication Attempts**

```kql
| where EventID == 4625
```

Filters for failed login events.

**4. Match IP Addresses With Geographic Records**

```kql
| evaluate ipv4_lookup(
    GeoIPDB_FULL,
    IpAddress,
    network
)
```

Matches source IP addresses against the network ranges contained in the GeoIP watchlist.

**5. Count Authentication Failures**

```kql
| summarize FailureCount = count()
    by IpAddress, latitude, longitude,
       cityname, countryname
```

Counts failed login events associated with each IP address and geographic location.

**6. Prepare Map Visualization Fields**

```kql
| project
    FailureCount,
    AttackerIp = IpAddress,
    latitude,
    longitude,
    city = cityname,
    country = countryname,
    friendly_location =
        strcat(cityname, " (", countryname, ")")
```

Prepares the coordinates, counts, and labels needed for the geographic visualization.

### Screenshot: Microsoft Sentinel Honeypot Attack Map

![Microsoft Sentinel Honeypot Attack Map](images/12-honeypot-attack-map.png)

[Back to Top](#table-of-contents)

---

# Security Findings

During the initial monitoring period of approximately three hours, the honeypot collected suspicious authentication activity.

The Microsoft Sentinel workbook displayed approximately **1,700 geographically mapped failed login events**.

The visualization identified failed authentication activity associated with four geographic locations.

### Initial Observations

| Observation | Result |
|---|---|
| Monitoring Duration | Approximately 3 hours |
| Primary Security Event | Event ID 4625 |
| Mapped Failed Login Events | Approximately 1,700 |
| Geographic Locations | 4 |
| GeoIP Reference Entries | Approximately 55,000 |
| Highest Observed Location | Wrexham, United Kingdom |

### Key Findings

- The honeypot received numerous failed authentication attempts within a short monitoring period.
- Windows Security Events provided useful information about authentication failures and their source IP addresses.
- Repeated failed logins may indicate automated credential guessing or brute-force activity.
- Log Analytics allowed centralized investigation of events collected from the VM.
- GeoIP enrichment made it possible to visualize approximate geographic origins.
- Microsoft Sentinel Workbooks provided a geographic representation of authentication activity.

> **Analyst Note:** Failed login events do not automatically confirm a malicious attack. Furthermore, IP geolocation provides approximate network locations and does not necessarily identify the attacker's physical location.

### Understanding the Different Counts

The GeoIP watchlist contained approximately 55,000 reference entries, while the workbook displayed approximately 1,700 mapped failed login events.

This difference is expected because the watchlist contains geographic reference information rather than authentication events.

The map displays only failed login events that meet the KQL filtering criteria and have matching geographic information.

---

# Challenges and Troubleshooting

## Challenge 1: Missing Geographic Information

**Problem:** Windows Security Events contained source IP addresses but did not directly provide geographic coordinates.

**Solution:** Imported a GeoIP CSV file into Microsoft Sentinel Watchlists and used the `ipv4_lookup()` KQL operator to associate source IP addresses with geographic records.

## Challenge 2: Workbook Query Parsing Error

**Problem:** Microsoft Sentinel initially displayed a KQL parsing error after a JSON configuration was entered directly into the query editor.

**Solution:** Used the appropriate workbook configuration editor for JSON and entered only KQL syntax into the query editor.

## Challenge 3: Different Watchlist and Attack Counts

**Problem:** The watchlist contained approximately 55,000 entries, while the geographic attack map displayed approximately 1,700 failed login events.

**Explanation:** Watchlist entries represent geographic reference data. The map displays actual recorded authentication failures that successfully match geographic information.

## Challenge 4: Centralizing Windows Security Events

**Problem:** Security Events were initially available only through the Windows VM's local Event Viewer.

**Solution:** Configured Azure Monitor Agent, the Windows Security Events connector, and a Data Collection Rule to collect and forward logs to Log Analytics.

---

# Lessons Learned

Completing this project improved my practical understanding of security monitoring and cloud-based SIEM operations.

### 1. Centralized Security Logging

I learned how to collect Windows Security Events from an Azure virtual machine and centralize them within a Log Analytics workspace.

Centralized logging allows security analysts to investigate events without accessing each individual machine.

### 2. SIEM Integration

I gained experience integrating Microsoft Sentinel with Azure Log Analytics and configuring Windows Security Events through Azure Monitor Agent.

This provided hands-on experience with cloud-native security monitoring infrastructure.

### 3. KQL Log Analysis

I learned how to use Kusto Query Language to filter security events, investigate authentication attempts, and identify IP addresses associated with repeated login failures.

### 4. Geographic Threat Visualization

I learned how to enrich Windows Security Events with GeoIP reference data and create a geographic attack map using Microsoft Sentinel Workbooks.

### 5. Honeypot Security Monitoring

I gained practical insight into how publicly exposed systems may receive automated connection attempts and repeated authentication failures.

### 6. Troubleshooting and Validation

I improved my ability to troubleshoot data collection, agent configuration, workbook queries, and geographic visualization issues.

---

# Lab Cleanup and Security

Because the honeypot was intentionally configured with permissive security settings, it should remain isolated from production systems.

After completing the experiment, the environment should be secured or decommissioned.

Recommended cleanup actions include:

- Stop and deallocate the Azure virtual machine.
- Remove unnecessary permissive inbound security rules.
- Re-enable Windows Defender Firewall if retaining the VM.
- Remove unused public IP addresses and other Azure resources.
- Review Log Analytics and Microsoft Sentinel usage.
- Monitor Azure Cost Management for additional charges.
- Delete unused lab resources when no longer needed.

**Important:** VM deallocation stops compute billing, but storage, networking, and security monitoring services may continue to generate charges.

---

# Conclusion

This project demonstrates the implementation of a cloud-based honeypot monitoring environment using Microsoft Azure and Microsoft Sentinel.

By deploying a Windows virtual machine, configuring Azure Monitor Agent, collecting Windows Security Events, analyzing authentication failures using KQL, and enriching security logs with GeoIP data, I created an interactive geographic attack map.

The project provided hands-on experience with several important cybersecurity concepts:

- Security Information and Event Management (SIEM)
- Security Event Collection
- Cloud Security Monitoring
- Log Analysis
- Threat Detection
- Kusto Query Language (KQL)
- Geographic Threat Intelligence Enrichment
- Security Data Visualization

These skills are directly applicable to Security Operations Center (SOC) operations and cloud security monitoring.

---

**Project Type:** Azure Security / Microsoft Sentinel / Honeypot / SIEM

**Focus Areas:** SOC Analysis, Threat Monitoring, Log Analytics, KQL, and Cloud Security
