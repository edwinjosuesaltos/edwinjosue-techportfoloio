# Azure CLI Ultimate Cheatsheet

Comprehensive Azure CLI reference including Accounts, VMs, Storage, Batch, Containers, IAM, CosmosDB, Monitoring, Web Apps, Key Vault, and Automation Scripts.

---

# Table of Contents

- [Installation](#installation)
- [Authentication & Account Management](#authentication--account-management)
- [Resource Groups](#resource-groups)
- [Virtual Machines](#virtual-machines)
- [Storage Accounts](#storage-accounts)
- [Batch Services](#batch-services)
- [Containers (ACS)](#containers-acs)
- [Web Apps & Function Apps](#web-apps--function-apps)
- [Key Vault](#key-vault)
- [IAM & RBAC](#iam--rbac)
- [Cosmos DB](#cosmos-db)
- [Monitoring & Alerts](#monitoring--alerts)
- [Azure CLI Configuration](#azure-cli-configuration)
- [PowerShell Equivalent Commands](#powershell-equivalent-commands)
- [Automation Scripts](#automation-scripts)

---

# Installation

| OS        | Command |
|-----------|----------|
| Windows   | Download MSI from https://aka.ms/installazurecliwindows |
| Mac       | `brew install azure-cli` |
| Linux     | `apt-get install azure-cli` |

Login:
```bash
az login
```

# Authentication & Account Management
## List Subscription
```bash
az account list
```
## Show current account context
```bash
az account show
```
## Login with service principal
```bash
az login --service-principal \
  --username <clientId> \
  --password <clientSecret> \
  --tenant <tenantId>
```
# Resource Groups
## Create resource group
```bash
az group create --name myResourceGroup --location eastus
```

## List resource groups
```bash
az group list -o table
```

## Delete resource group
```bash
az group delete --name myResourceGroup
```

## List resources in group
```bash
az resource list -g myResourceGroup -o table
```

# Virtual Machines
## List VMs
```bash
az vm list -o table
```
## List VM sizes
```bash
az vm list-sizes --location eastus
```

## Create Linux VM
```bash
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Ubuntu2204
```
## Create Windows VM
```bash
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Win2022Datacenter
```
## Start / Stop / Restart
```bash
az vm start --resource-group myResourceGroup --name myVM
az vm stop --resource-group myResourceGroup --name myVM
az vm restart --resource-group myResourceGroup --name myVM
```
## Deallocate
```bash
az vm deallocate --resource-group myResourceGroup --name myVM
```

## Delete VM
```bash
az vm delete --resource-group myResourceGroup --name myVM
```
# Storage Accounts
## Create Storage Account
```bash
az storage account create \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS
```
## List Storage Accounts
```bash
az storage account list -o table
```
# Batch Services
## Create batch account
```bash
az batch account create \
  -g myResourceGroup \
  -n myBatchAccount \
  -l eastus
```
## Login to batch account
```bash
az batch account login -g myResourceGroup -n myBatchAccount
```
## Create pool
```bash
az batch pool create \
  --id mypool \
  --vm-size Standard_A1 \
  --image canonical:ubuntuserver:20.04-LTS \
  --node-agent-sku-id "batch.node.ubuntu 20.04"
```
## Resize pool
```bash
az batch pool resize --pool-id mypool --target-dedicated 3
```
## Create job
```bash
az batch job create --id myjob --pool-id mypool
```
# Containers (ACS)
## Create cluster
```bash
az acs create \
  -n acs-cluster \
  -g myResourceGroup \
  -d mydns \
  --generate-ssh-keys
```
## List clusters
```bash
az acs list -o table
```
## Scale cluster
```bash
az acs scale \
  -g myResourceGroup \
  -n acs-cluster \
  --new-agent-count 4
```
# Web Apps & Function Apps
## List Web Apps
```bash
az webapp list -o table
```
## View app settings
```bash
az webapp config appsettings list \
  -g myResourceGroup \
  -n myWebApp
```
## Deploy ZIP
```bash
az webapp deployment source config-zip \
  -g myResourceGroup \
  -n myWebApp \
  --src publish.zip
```

# Key Vault
## List Secrets
```bash
az keyvault secret list --vault-name myVault
```
## Show secret
```bash
az keyvault secret show \
  --vault-name myVault \
  --name mySecret
```
# IAM & RBAC
## List role assignments (RG)
```bash
az role assignment list \
  --resource-group myResourceGroup \
  -o table
```

## List role assignments (resource)
```bash
az role assignment list \
  --scope <resource-id> \
  -o table

```
# Cosmos DB
## List Cosmos DB accounts

## List by subscription

# Monitoring & Alerts
## Create CPU alert
## List alerts



# Azure CLI Configuration
## Set default location
## Set default resource group
## Enable parameter persistence

# PowerShell Equivalent Commands
## Set context
## Get context
## Modify custom RBAC role


# Automation Scripts
## Trim CR in WSL
## Loop through subscriptions


