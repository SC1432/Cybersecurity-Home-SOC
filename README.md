# Azure SOC Home Lab — Honeypot Threat Detection with Microsoft Sentinel

## Overview

This project is a self-built Security Operations Center (SOC) home lab hosted on Microsoft Azure. A Windows Server virtual machine was deliberately exposed to the internet as a "honeypot" to attract real-world attack traffic. Security telemetry from the VM was piped into Microsoft Sentinel (Azure's cloud-native SIEM) for log analysis, threat hunting, and geographic visualization of attacker activity using KQL (Kusto Query Language).

The goal of the project was to gain hands-on experience with the full pipeline of a SOC analyst's workflow: infrastructure setup, log ingestion, detection engineering, and threat intelligence enrichment — all within a real cloud environment rather than a simulated one.

## Architecture

- **Azure Resource Group** — container for all lab resources
- **Virtual Network (VNet)** — network boundary for the honeypot VM
- **Windows Server Virtual Machine** — the honeypot itself, intentionally exposed
- **Log Analytics Workspace** — central store for collected security event logs
- **Microsoft Sentinel** — SIEM layer deployed on top of the Log Analytics Workspace
- **Data Collection Rule (DCR)** + **Azure Monitor Windows Agent (AMA)** — pipeline forwarding Windows Event Viewer logs from the VM into Sentinel
- **Sentinel Watchlist** — CSV-based IP-to-geolocation lookup table used to enrich log data

## Build Process

### 1. Environment Setup
Provisioned an Azure for Students subscription and built the core infrastructure: resource group → virtual network → virtual machine.

### 2. Creating the Honeypot
Configured the VM to be attractive and reachable by real attackers:
- Added a permissive inbound Network Security Group (NSG) rule (any source, port, protocol, destination)
- Disabled all Windows Defender Firewall profiles
- Verified external reachability by pinging the VM from an external network

### 3. Log Collection Pipeline
- Monitored Windows Event Viewer and observed a high volume of audit failure logs, confirming active attack attempts against the exposed VM
- Created a Log Analytics Workspace and deployed Microsoft Sentinel on top of it
- Built a Data Collection Rule to forward Windows Security Event logs from the VM into the workspace

### 4. Troubleshooting
- Initial KQL queries returned zero results despite an active honeypot
- Root-caused the issue to a Sentinel Content Hub UI loading failure, which prevented the required **Windows Security Events** solution from installing
- Installed the solution via Content Hub, then rebuilt the Data Connector-based DCR
- Diagnosed the difference between the two DCRs by inspecting resource associations — the Content Hub-based DCR was correctly linked to the VM as a resource, auto-tagged `createdBy: Sentinel`, and auto-installed the AzureMonitorWindowsAgent extension on the VM

### 5. Threat Hunting & Enrichment
- Queried ingested logs using **KQL** to isolate specific security events (e.g., Event ID 4625 — failed logon attempts)
- Extracted attacker IP addresses from log events and performed IP geolocation lookups
- Uploaded a prebuilt IP-range-to-geolocation CSV as a Sentinel **Watchlist**, enabling enrichment via `_GetWatchlist()` in KQL
- Wrote a KQL query joining failed logon events against the watchlist via `ipv4_lookup()` to resolve attacker geolocation

### 6. Visualization
Built a custom Sentinel Workbook panel using a JSON-defined KQL query to render a **heatmap of attacker origin by geolocation**, aggregating failed login attempts by IP, city, and country.

## Sample KQL Query

```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where IpAddress == "<attacker IP address>"
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
```

## Skills Demonstrated

- **Cloud infrastructure provisioning** — Azure resource groups, virtual networks, VMs, NSGs
- **SIEM deployment & configuration** — Microsoft Sentinel, Log Analytics Workspaces, Data Collection Rules
- **Log ingestion pipelines** — Azure Monitor Agent, Windows Event Viewer log forwarding
- **Threat hunting & detection** — KQL query writing against live security event data
- **Threat intelligence enrichment** — Watchlists, IP geolocation lookups
- **Data visualization** — custom Sentinel Workbooks (JSON-defined heatmap panel)
- **Troubleshooting & root-cause analysis** — diagnosing a broken Content Hub UI and stale documentation, then independently resolving via resource inspection
- **Security fundamentals** — offensive exposure setup (honeypot design), defensive monitoring, and analysis of real-world attack telemetry (brute-force login attempts, audit failures)

## Next Steps

Planned extensions to the lab include:
- **SOAR automation** — building automated triage playbooks in Sentinel
- **Custom Analytics Rules** — proactive alerting on defined detection logic
- **Expanded attack surface** — adding a Linux honeypot for cross-platform telemetry
- **Threat Intelligence integration** — ingesting external Indicators of Compromise (IoCs) feeds

## Notes

This lab was built iteratively while following an Azure/Sentinel tutorial that had gone out of date due to portal UI changes (e.g., Data Connectors relocating under Configuration, and connectors requiring a Content Hub solution install as a prerequisite). Adapting to these discrepancies became part of the learning process itself.
