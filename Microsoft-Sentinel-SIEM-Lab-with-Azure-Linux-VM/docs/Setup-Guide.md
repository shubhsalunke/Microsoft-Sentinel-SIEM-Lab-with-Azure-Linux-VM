# Step-by-Step Lab Setup Guide

This guide walks you through provisioning the virtual infrastructure needed for the Microsoft Sentinel SIEM Lab.

---

## 🛠️ Step 1: Azure Authentication and Preparation

First, authenticate to your Azure account using the Azure CLI. If you are using a headless environment or secondary shell, `--use-device-code` makes it easy to authorize via browser.

```powershell
# Authenticate to Azure
az login --use-device-code

# (Optional) Verify your active subscription
az account show --output table
```

Ensure your CLI session is targeting the correct subscription. If you have multiple subscriptions, set the correct one:
```powershell
az account set --subscription "YOUR-SUBSCRIPTION-ID"
```

---

## 📂 Step 2: Resource Group Creation

Create a Resource Group which will act as a logical container for your virtual machine, workspace, network settings, and security policies.

```powershell
az group create \
  --name rg-shubh \
  --location centralindia
```
*Note: You can change the location to any valid Azure region (e.g., `eastus`, `westeurope`, `southeastasia`).*

---

## 📊 Step 3: Log Analytics Workspace (LAW)

Deploy the database workspace to store and aggregate logs.

```powershell
az monitor log-analytics workspace create \
  --resource-group rg-shubh \
  --workspace-name log-shubh \
  --location centralindia
```

---

## 💻 Step 4: Deploying the Azure Linux VM

Generate or locate your SSH key pair. Under Windows, keys are typically stored at `C:\Users\<YourUsername>\.ssh\`.

Run the following command to provision a virtual machine running Ubuntu 24.04:

```powershell
az vm create \
  --resource-group rg-shubh \
  --name vm-shubh \
  --image Ubuntu2404 \
  --admin-username azureuser \
  --ssh-key-values C:\Users\USERNAME\.ssh\mujahed.pub
```

### Key CLI Parameters Explained:
- `--image Ubuntu2404`: Installs the latest Canonical Ubuntu 24.04 LTS image.
- `--admin-username azureuser`: Sets the default administrator account username.
- `--ssh-key-values`: Points to your public key (`.pub`) file. Azure injects this key into `/home/azureuser/.ssh/authorized_keys` for secure authentication.

---

## 🔒 Step 5: Network Configuration & Security

To access your VM, you must explicitly allow SSH traffic through the Network Security Group (NSG).

```powershell
# Allow inbound Port 22 (SSH)
az vm open-port \
  --resource-group rg-shubh \
  --name vm-shubh \
  --port 22
```

### Retrieving VM Public IP
To connect to the VM, obtain its public IP address:
```powershell
az vm list-ip-addresses --resource-group rg-shubh --name vm-shubh --output table
```

---

## 🔌 Step 6: Connect via SSH

Use PowerShell, command prompt, or Git Bash to connect:

```powershell
ssh -i "C:\Users\USERNAME\.ssh\mujahed.pem" azureuser@<YOUR-VM-PUBLIC-IP>
```

> [!WARNING]
> **Key Permissions Error (Windows)**: If you get a "permissions are too open" error for your private key (`.pem`), run the following commands in PowerShell to restrict permissions:
> ```powershell
> icacls "C:\Users\USERNAME\.ssh\mujahed.pem" /inheritance:r
> icacls "C:\Users\USERNAME\.ssh\mujahed.pem" /grant:r "${env:USERNAME}:(R)"
> ```

---

## 🛠️ Infrastructure Troubleshooting

### 1. VM Deployment Hangs or Fails
If VM creation fails due to SKU constraints in the selected region:
- Check alternative size SKUs (default is Standard_D2s_v3).
- Try changing the region by updating `--location` to `eastus2` or `swedencentral`.

### 2. SSH Timeouts
If you cannot establish an SSH connection:
- Verify that the port is open: `az vm open-port --resource-group rg-shubh --name vm-shubh --port 22`.
- Check if your local firewall or network restricts outbound SSH traffic (port 22).
- Retrieve your VM's operational status:
  ```powershell
  az vm get-instance-view --resource-group rg-shubh --name vm-shubh --query "instanceView.statuses[1]" --output table
  ```
