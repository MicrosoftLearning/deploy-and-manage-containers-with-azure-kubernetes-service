---
Exercise:
  title: 'Exercise: Review the deployment and deprovision all resources'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Exercise 4 - Deploy applications to Azure Kubernetes Service (AKS)

## Objectives

This guided project consist of the following exercises:

+ Exercise 1: Provision Azure Container Registry (ACR) and Azure Kubernetes Service (AKS)
+ Exercise 2: Build a Linux and Windows container images and store them in ACR
+ Exercise 3: Deploy container images to AKS
+ **Exercise 4: Review the deployment and deprovision all resources**

In this exercise you review the deployment of the services in the previous exercises and deprovision all the resources.

# Exercise 4: Review the deployment and deprovision all resources
In this exercise, you will review the results of the deployments and deprovision all resources.

>**Note**: To complete this exercise you will need an [Azure subscription](https://azure.microsoft.com/free/).
> For any properties that are not specified, use the default value.

### Task 1: Review the AKS deployments and services
In this task, you will review results of both deployments, including the deployments and services objects.

1. From the Bash session of Azure Cloud Shell, display the status of both deployments by running the following commands:

   ```kubectl
   kubectl get deployments -n=dev-node
   kubectl get deployments -n=dev-dotnet
   ```

   > **Note:** Before you proceed to the next step, verify that both deployments are listed with the ready status. If that's not the case, wait for another minute, rerun the two aforementioned commands, and check the deployment status again.

1. From the Bash session of Azure Cloud Shell, display the status of the two services which were included in the manifest files by running the following commands:

   ```kubectl
   kubectl get services -n=dev-node
   kubectl get services -n=dev-dotnet
   ```

1. Verify that the listing for each service includes a value in the **EXTERNAL-IP** column. 
1. Use a web browser to navigate to the IP addresses you identified in the previous step and verify that the resulting web pages display the **Hello World from Node** and **Hello World from .Net 7** messages, respectively.

### Task 2: Delete all resources
In this task, you will delete all resources provisioned in this lab.

1. From the Bash session of Azure Cloud Shell, display the listing of resources in the two resource groups provisioned in this exercise by running the following commands:

   ```azurecli
   az resource list --resource-group 'acr-01-RG' --query "[].name" --output tsv
   az resource list --resource-group 'aks-01-RG' --query "[].name" --output tsv
   ```

   > **Note:** Verify that these are the resources you want to delete. If so, proceed to the next step.

1. From the Bash session of Azure Cloud Shell, delete all resources provisioned in this exercise by running the following commands:

   ```azurecli
   az group delete --name 'acr-01-RG' --no-wait --yes
   az group delete --name 'aks-01-RG' --no-wait --yes
   ```

   > **Note:** The command executes asynchronously (as enforced by the --nowait parameter), so even both commands return immediately to the Bash shell prompt, it will take a few minutes before the resource groups and their resources are actually removed.

1. Close the Azure Cloud Shell pane.
