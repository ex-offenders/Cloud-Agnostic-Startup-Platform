# Deploying Kubernetes Cluster on Azure Cloud

This directory contains the concise code sample illustrating our deployment of Kubernetes on Azure, leveraging Terragrunt to maintain the Don't Repeat Yourself (DRY) principle in our codebase

### Storing States
Before we proceed with creating the resources with terragrunt, we need to create a storange account manually to store the terraform states. 

```
az group create --name terraform-state --location westus
az storage account create --name exoffenderstfstate --resource-group terraform-state
```
