---
page_type: sample
languages:
- json
products:
- azure-resource-manager
description: "data-science-sandbox defines an Azure-hosted sandbox environment to enable third-parties to collaborate on data science solutions over protected data"
urlFragment: "azure-data-science-sandbox"
---

# data-science-sandbox

![MIT license badge](https://img.shields.io/badge/license-MIT-green.svg)

`data-science-sandbox` defines an Azure-hosted sandbox environment to enable third-parties to collaborate on data science solutions over protected data sets

## Contents

| File/folder       | Description                                |
|-------------------|--------------------------------------------|
| `azuredeploy.json`| Azure Resource Manager template to deploy the sandbox.                        |
| `README.md`       | This README file.                          |
| `LICENSE`         | The license for the sample.                |

## Prerequisites

* Install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

## Setup

1. Clone or download this repository
2. Run the following command to retrieve the `objectId` for your AAD user account

```shell
az ad user show --id $(az account show --query user.name -o tsv) --query objectId -o tsv
```

## Running the sample

```shell
az group create -n sandbox -l westus2
az group deployment create -g sandbox --template-file azuredeploy.json --parameters sandboxOwnerObjectId=<objectId>
```

## Key concepts

![architecture](docs/architecture.png)

The sandbox is modeled as an Azure DevTestLab that sits inside an isolated VNET. After deploying the lab, admins can customize the environment, while sandbox operators can create new VMs to enable experimentation while keeping data secure.

### Roles

It's likely that a few distinct personas may interact with the sandbox

* Administrator:
  * Deploys the ARM template to create the sandbox
  * Ensures compliance with organizational IT policy
  * Works with the Sandbox Operator to enable allowed resources (Azure resources, web services, etc)
  * Links the sandbox to any outside resources (storage, networking, etc)
  * Grants access to the Sandbox Operator
* Sandbox Operator:
  * Creates sandbox VMs
  * Grants access to Data Scientists
  * Provides resources for the sandbox (blobs, secrets, etc)
* Data Scientist:
  * Perform data science & machine learning tools inside sandbox VMs

### Architecture

There are a number of notable design decisions in the base template. You may choose to enhance or drop any of these, but they provide a safe default environment to start from

* DevTestLabs
  * Limited to defined VM images and sizes
  * Public artifact feeds disabled
  * Predefined Windows & Linux DSVM formulas for instant provisioning
  * VMs are created in a defined resource group & VNET
* Virtual Machines
  * Based on the Data Science Virtual Machine image (Windows 2019 or Ubuntu)
  * Must be accessed via Azure Bastion
  * Auto-shutdown enabled to save on costs
  * Managed Identity enabled to access Azure resources w/o credentials
* Storage
  * Accessible only over explicitly defined networks
  * Always-encrypted via Microsoft-managed keys
  * Secure access (HTTPS) enforced
  * VM Managed Identity is granted read-only access
* Networking
  * Inbound & outbound traffic must be explicitly added to the NSG allow list
  * Services preapproved via NSG Service Tag:
    * AzureActiveDirectory - for AAD logon (Azure Storage Explorer, Visual Studio, Azure CLI)
    * AzureResourceManager - to access ARM & enumerate what resources are available (Storage, Key Vault)
  * Private DNS enables resolution of private endpoints
* Key Vault
  * Provides a secure area to store sandbox-level shared secrets
  * Preloaded with a minimum-scoped read-only SAS URI for applications that don't support Managed Identity

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
