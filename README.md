# Microsoft Sentinel SIEM Lab with Azure Linux VM

## Objective

This lab demonstrates how to:

- Create an Azure Linux Virtual Machine
- Create a Log Analytics Workspace
- Enable Microsoft Sentinel
- Install Azure Monitor Agent (AMA)
- Configure Syslog Data Collection Rule (DCR)
- Send Linux Syslog events to Microsoft Sentinel
- Verify Heartbeat and Syslog logs using KQL queries

---

# Project Structure

```text
Microsoft-Sentinel-SIEM-Lab/
│
├── README.md
│
├── docs/
│   ├── Architecture.md
│   ├── Setup-Guide.md
│   └── Sentinel-Configuration.md
│
├── queries/
│   ├── heartbeat.kql
│   ├── syslog.kql
│   ├── siem-test.kql
│   └── auth-logs.kql
│
├── screenshots/
│   ├── 01-log-analytics.png
│   ├── 02-sentinel.png
│   ├── 03-heartbeat.png
│   ├── 04-syslog.png
│   └── 05-siem-test.png
│
└── LICENSE
```

---

# Architecture

```text
                Azure Linux VM
                       │
                       │
             Azure Monitor Agent (AMA)
                       │
                       │
          Data Collection Rule (DCR)
                       │
                       │
          Log Analytics Workspace
                       │
                       │
             Microsoft Sentinel
                       │
                       │
                KQL Queries & Alerts
```

---

# Prerequisites

- Azure Subscription
- Azure CLI
- SSH Key Pair
- Ubuntu Linux VM

---

# Step 1 - Login to Azure

```powershell
az login --use-device-code
```

---

# Step 2 - Create Resource Group

```powershell
az group create \
--name rg-shubh \
--location centralindia
```

---

# Step 3 - Create Log Analytics Workspace

```powershell
az monitor log-analytics workspace create \
--resource-group rg-shubh \
--workspace-name log-shubh \
--location centralindia
```

---

# Step 4 - Enable Microsoft Sentinel

Azure Portal

```
Log Analytics Workspace
        ↓
Microsoft Sentinel
        ↓
Create
```

Workspace

```
log-shubh
```

Click

```
Create
```

---

# Step 5 - Create Azure Linux VM

```powershell
az vm create \
--resource-group rg-shubh \
--name vm-shubh \
--image Ubuntu2404 \
--admin-username azureuser \
--ssh-key-values C:\Users\USERNAME\.ssh\mujahed.pub
```

---

# Step 6 - Open SSH Port

```powershell
az vm open-port \
--resource-group rg-shubh \
--name vm-shubh \
--port 22
```

---

# Step 7 - Connect to VM

```powershell
ssh -i "C:\Users\USERNAME\.ssh\mujahed.pem" azureuser@PUBLIC-IP
```

---

# Step 8 - Install Azure Monitor Agent

Azure Portal

```
Virtual Machine
        ↓
Extensions + Applications
        ↓
Add
        ↓
Azure Monitor Linux Agent
        ↓
Install
```

Wait until

```
Provisioning State : Succeeded
```

---

# Step 9 - Configure Syslog Connector

```
Microsoft Sentinel
        ↓
Data Connectors
        ↓
Syslog via AMA
        ↓
Open Connector Page
        ↓
Create Data Collection Rule
```

---

# Step 10 - Configure Data Collection Rule

Rule Name

```
syslog-dcr-shubh
```

Resource

```
vm-shubh
```

Configure

| Facility | Minimum Log Level |
| ---------------- | ---------------- |
| LOG_AUTH | LOG_DEBUG |
| LOG_AUTHPRIV | LOG_DEBUG |
| LOG_DAEMON | LOG_DEBUG |
| LOG_KERN | LOG_DEBUG |
| LOG_SYSLOG | LOG_DEBUG |
| LOG_USER | LOG_DEBUG |

Click

```
Review + Create
```

---

# Step 11 - Generate Test Logs

```bash
logger "SIEM TEST 100"
logger "SIEM TEST 200"
logger "SIEM TEST 300"
```

---

# Step 12 - Verify Heartbeat

```kusto
Heartbeat
| sort by TimeGenerated desc
```

---

# Step 13 - Verify Syslog

```kusto
Syslog
| take 10
```

---

# Step 14 - Verify Custom SIEM Logs

```kusto
Syslog
| where SyslogMessage contains "SIEM TEST"
| sort by TimeGenerated desc
```

---

# Test Queries

## Heartbeat

```kusto
Heartbeat
| take 10
```

## Syslog

```kusto
Syslog
| take 10
```

## Custom Logger

```kusto
Syslog
| where SyslogMessage contains "SIEM TEST"
```

## Authentication Logs

```kusto
Syslog
| where Facility == "auth"
or Facility == "authpriv"
```

---

# Architecture Flow

```text
Linux VM
    │
    ▼
Azure Monitor Agent (AMA)
    │
    ▼
Data Collection Rule (DCR)
    │
    ▼
Log Analytics Workspace
    │
    ▼
Microsoft Sentinel
    │
    ▼
Kusto Query Language (KQL)
    │
    ▼
Security Monitoring & Threat Detection
```

---

# Lab Status

| Component | Status |
|-----------------------------|----------|
| Azure VM | ✅ |
| SSH Connection | ✅ |
| Azure Monitor Agent | ✅ |
| Log Analytics Workspace | ✅ |
| Microsoft Sentinel | ✅ |
| Data Collection Rule | ✅ |
| Heartbeat Logs | ✅ |
| Syslog Logs | ✅ |
| SIEM Test Logs | ✅ |

---

# Result

Successfully deployed a Microsoft Sentinel SIEM environment on Azure using an Ubuntu Linux Virtual Machine, Azure Monitor Agent (AMA), Log Analytics Workspace, and Data Collection Rules (DCR). Verified Heartbeat, Syslog, and custom Linux events using Kusto Query Language (KQL), enabling centralized security monitoring and log analysis.
