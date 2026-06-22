# Azure_PackerMan1577

## Overview

Azure_PackerMan1577 demonstrates how to use **HashiCorp Packer** with **Microsoft Azure** to create reusable Virtual Machine Images and deploy multiple Virtual Machine instances from a single golden image.

This project follows Infrastructure as Code (IaC) principles and automates the image creation process, ensuring consistency, scalability, and repeatability across Azure environments.

---

## What is Packer?

Packer is an open-source tool developed by HashiCorp that allows users to create identical machine images for multiple platforms from a single source configuration.

Using Packer with Azure enables:

* Automated image creation
* Consistent VM deployments
* Reduced configuration drift
* Faster infrastructure provisioning
* Scalable cloud environments

---

## Architecture

```text
+---------------------+
|  Packer Template    |
+----------+----------+
           |
           v
+---------------------+
| Azure Build VM      |
| (Temporary VM)      |
+----------+----------+
           |
           v
+---------------------+
| Azure Managed Image |
+----------+----------+
           |
           v
+---------------------+
| Multiple Azure VMs  |
+---------------------+
```

---

## Prerequisites

Before getting started, ensure you have:

### Azure Requirements

* Azure Subscription
* Resource Group
* Azure CLI
* Contributor Permissions

### Local Requirements

* Packer (Latest Version)
* PowerShell or Bash
* Git
* Visual Studio Code (Optional)

---

## Installing Packer

### Windows

Download from:

https://developer.hashicorp.com/packer/downloads

Verify installation:

```bash
packer version
```

Expected Output:

```bash
Packer v1.x.x
```

---

## Azure Authentication

Login to Azure:

```bash
az login
```

Verify account:

```bash
az account show
```

---

## Creating a Resource Group

```bash
az group create \
--name AzurePackerRG \
--location eastus
```

---

## Creating Service Principal

Packer requires authentication to Azure.

```bash
az ad sp create-for-rbac \
--name AzurePackerSP \
--role Contributor \
--scopes /subscriptions/<subscription-id>
```

Output:

```json
{
  "appId": "",
  "displayName": "",
  "password": "",
  "tenant": ""
}
```

Save these values securely.

---

## Project Structure

```text
Azure_PackerMan1577
│
├── images/
│   └── azure-image.pkr.hcl
│
├── scripts/
│   ├── install-tools.ps1
│   └── configure-vm.ps1
│
├── variables.pkrvars.hcl
│
├── README.md
│
└── .gitignore
```

---

## Sample Packer Configuration

```hcl
packer {
  required_plugins {
    azure = {
      source  = "github.com/hashicorp/azure"
      version = ">= 2.0.0"
    }
  }
}

source "azure-arm" "windows" {

  subscription_id = var.subscription_id
  client_id       = var.client_id
  client_secret   = var.client_secret
  tenant_id       = var.tenant_id

  managed_image_resource_group_name = "AzurePackerRG"
  managed_image_name                = "GoldenImage"

  os_type         = "Windows"
  image_publisher = "MicrosoftWindowsServer"
  image_offer     = "WindowsServer"
  image_sku       = "2022-datacenter"

  location = "East US"

  vm_size = "Standard_D2s_v3"
}

build {
  sources = [
    "source.azure-arm.windows"
  ]

  provisioner "powershell" {
    script = "scripts/configure-vm.ps1"
  }
}
```

---

## Validate Template

```bash
packer init .
```

```bash
packer validate .
```

---

## Build Azure Image

```bash
packer build .
```

Packer will:

1. Create a temporary build VM
2. Install required software
3. Execute provisioning scripts
4. Capture the VM
5. Create a Managed Image
6. Remove temporary resources

---

## Deploy VM from Managed Image

```bash
az vm create \
--resource-group AzurePackerRG \
--name VM01 \
--image GoldenImage \
--admin-username azureuser \
--generate-ssh-keys
```

---

## Creating Multiple VM Instances

### Bash

```bash
for i in {1..5}
do
  az vm create \
  --resource-group AzurePackerRG \
  --name vm-$i \
  --image GoldenImage \
  --admin-username azureuser \
  --generate-ssh-keys
done
```

### PowerShell

```powershell
1..5 | ForEach-Object {
    az vm create `
    --resource-group AzurePackerRG `
    --name "vm$_" `
    --image GoldenImage `
    --admin-username azureuser
}
```

---

## Benefits of Golden Images

* Faster VM Deployment
* Consistent Environments
* Reduced Provisioning Time
* Easier Scaling
* Improved Security
* Standardized Infrastructure

---

## Common Commands

### Initialize Packer

```bash
packer init .
```

### Format Template

```bash
packer fmt .
```

### Validate Template

```bash
packer validate .
```

### Build Image

```bash
packer build .
```

### List Images

```bash
az image list --output table
```

---

## Security Best Practices

* Never hardcode secrets
* Use Azure Key Vault
* Rotate Service Principal credentials regularly
* Apply least-privilege access
* Store sensitive values in environment variables

Example:

```bash
export ARM_CLIENT_ID=""
export ARM_CLIENT_SECRET=""
export ARM_SUBSCRIPTION_ID=""
export ARM_TENANT_ID=""
```

---

## Troubleshooting

### Authentication Failed

Verify:

```bash
az login
```

and Service Principal credentials.

### Image Build Timeout

Increase timeout values inside the Packer configuration.

### Provisioner Failure

Review build logs:

```bash
PACKER_LOG=1 packer build .
```

---

## Learning Resources

* Microsoft Azure Virtual Machines
* HashiCorp Packer Documentation
* Azure Managed Images
* Azure Shared Image Gallery
* Azure Resource Manager (ARM)

---

## Future Enhancements

* Azure Shared Image Gallery Integration
* Terraform Deployment Automation
* GitHub Actions CI/CD
* Azure DevOps Pipelines
* Automated Security Hardening
* Custom Windows and Linux Images

---

## Author

**Pranav Gor**

MCA Student | Cloud Computing Enthusiast | Future Azure Administrator

---

## License

This project is intended for educational and learning purposes.
