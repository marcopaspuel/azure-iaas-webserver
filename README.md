# Azure Infrastructure Operations Project
Deploying a scalable IaaS web server in Azure.

### Introduction
This project, contains a Packer template, and a Terraform template to deploy a customizable, scalable web server in Azure.
It uses Packer to create the server image, and Terraform for deploying a scalable cluster of servers—with a load balancer
to manage the incoming traffic. It also adheres to the security best practices ensuring that the infrastructure is secure.

![pycharm1](project_architecture.png)

### Prerequisites
- Create an [Azure Account](https://portal.azure.com)
- Install the [Azure command line interface](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- Install [Packer](https://www.packer.io/downloads)
- Install [Terraform](https://www.terraform.io/downloads.html)

### Getting Started

1. Clone this repository
2. Deploy azure policies following [these instructions](azure-policies/README.md)
3. Deploy a scalable web server following the [instruction](#instructions) bellow:

### Instructions
- [1. Create a Service Principle for Packer and Terraform](#1-create-a-service-principle-for-packer-and-terraform)
- [2. Create a Resource Group for the Packer image](#2-create-a-resource-group-for-the-packer-image)
- [3. Deploy the packer image](#3-deploy-the-packer-image)
- [4. Deploy the infrastructure with Terraform](#4-deploy-the-infrastructure-with-terraform)

#### 1. Create a Service Principle for Packer and Terraform
Log into your Azure account
``` bash
    az login 
```

``` bash 
    az account set --subscription="SUBSCRIPTION_ID"
```
Create Service Principle
``` bash
    az ad sp create-for-rbac --name azure-iaas-webserver --role="Contributor" --scopes="/subscriptions/SUBSCRIPTION_ID"
```

This command will output 5 values:
``` json
{
  "appId": "00000000-0000-0000-0000-000000000000",
  "displayName": "azure-cli-2017-06-05-10-41-15",
  "name": "http://azure-cli-2017-06-05-10-41-15",
  "password": "0000-0000-0000-0000-000000000000",
  "tenant": "00000000-0000-0000-0000-000000000000"
}
``` 
Create a .env.sh file inside the packer directory and copy the content of the .env.sh.template to the newly created file.
Change the parameters based on the output of the previous command. These values map to the .evn.sh variables like so:

    appId is the ARM_CLIENT_ID
    password is the ARM_CLIENT_SECRET
    tenant is the ARM_TENANT_ID

For more information about *Authenticating to Azure using a Service Principal and a Client Secret*
(follow this [Guide](https://www.terraform.io/docs/providers/azurerm/guides/service_principal_client_secret.html))

#### 2. Create a Resource Group for the Packer image
Create Resource Group
``` bash
    az group create -l "LOCATION" -n "RESOURCE_GROUP_NAME" --tags Project=iaas-webserver
```
Ensure that the *location* and *resource group* that you specify here is the same specified in [server.json](packer/server.json).

#### 3. Deploy the packer image
Source environment variables 
```
    source packer/.env.sh
```
Run packer file
```
    packer build ./packer/server.json
```
This will create a packer image in the *resource group* specified in the previous step.

#### 4. Deploy the infrastructure with Terraform
Edit variables in the [variables.tf](terraform/variables.tf) to reflect your desired infrastructure.

The following items should be updated accordingly:

- prefix
- location (should match packer image location)
- username
- password
- image_id (SUBSCRIPTION_ID, RESOURCE_GROUP_NAME, and IMAGE_NAME)
- instance_count

Run Terraform plan 
``` bash
    cd terraform/
```
``` bash
    terraform init
```
``` bash
    terraform plan -out solution.plan
```
After running the plan you should see all the resources that will be created.

Run Terraform apply
``` bash
    terraform apply "solution.plan"
```

If everything runs correctly you should be able to see something like the screenshot bellow:

![pycharm3](terraform-apply-output.png)

### Output
Service Principal with permissions to manage resources in the specified Subscription:

- [Azure Service Principal](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps)

Packer creates the following resources:

- [Image resource group](https://portal.azure.com/#blade/HubsExtension/BrowseResourceGroups)
- [Managed virtual machine image](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Compute%2Fimages)

Terraform creates the following resources:

- Availability Set
- Azure Managed Disk(s)
- Load Balancer
- Network Interface Card(s)
- Network Security Group
- Public IP
- Virtual Machine(s)

All can be found under the specified [resource group](https://portal.azure.com/#blade/HubsExtension/BrowseResourceGroups).


### Clean Up
To delete all the resources created by terraform you can use the following command:
``` bash
    terraform destroy
```
To delete the packer image run the following command:
``` bash
    az image delete -g "RESOURCE_GROUP_NAME" -n "IMAGE_NAME"
```
To delete the resource group run the following command:
``` bash
    az group delete --no-wait --name "RESOURCE_GROUP_NAME"
```
To delete the Service Principal Created in step 1 run the following command:
``` bash
    az ad sp delete --id 00000000-0000-0000-0000-000000000000
```
