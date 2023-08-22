---
Exercise:
  title: 'Exercise: Deploy container images to Azure Kubernetes Service'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Exercise 3 - Deploy applications to Azure Kubernetes Service (AKS)

## Objectives

This guided project consist of the following exercises:

+ Exercise 1: Provision Azure Container Registry (ACR) and Azure Kubernetes Service (AKS).
+ Exercise 2: Build a Linux and Windows container images and store them in Azure Container Registry.
+ **Exercise 3: Deploy container images to Azure Kubernetes Service.**
+ Exercise 4: Review the deployment and deprovision all resources.

In this exercise you deploy container images to Azure Kubernetes Service.

## Exercise 3: Deploy container images to AKS 
In this exercise, you will deploy two container images you created earlier in this exercise to the AKS cluster.
>**Note**: To complete this exercise you will need an [Azure subscription](https://azure.microsoft.com/free/).
> For any properties that are not specified, use the default value.
> **Note:** Before you proceed with this exercise, make sure that the provisioning of the AKS cluster has successfully completed.

### Task 1: Create custom AKS namespaces
In this task, you will create two namespaces on the AKS cluster you created earlier in this lab.

1. In the Azure portal, in the **Search** text box, search for and select **Kubernetes services**.
1. On the **Kubernetes services** page, select **aks-01**.
1. On the **aks-01** page, in the vertical hub menu, select **Namespaces**.
1. On the **aks-01 \| Namespaces** page, select **+ Create** and, in the dropdown menu, select **Namespace**.
1. On the **Create a namespace** pane, in the **Name** textbox, enter **dev-node** and select **Create**.
1. On the **aks-01 \| Namespaces** page, select **+ Create** and, in the dropdown menu, select **Namespace**.
1. On the **Create a namespace** pane, in the **Name** textbox, enter **dev-dotnet** and select **Create**.

### Task 2: Create a Kubernetes manifest for deploying the Linux image
In this task, you will create a Kubernetes manifests for deploying the Linux image to the Linux node pool

1. In the Azure portal, select the **Cloud Shell** icon.
1. Ensure that **Bash** appears in the drop-down menu in the upper-left corner of the Cloud Shell pane.
1. From the Bash session of Azure Cloud Shell, create a directory that will host the deployment manifest for provisioning pods based on the Linux image and change to it from the current directory by running the following commands:

   ```bash
   mkdir ~/aks-l01
   cd ~/aks-l01
   ```

1. In the Bash session of Azure Cloud Shell, use the built-in editor to create a file named aks-deployment-l01.yaml in the aks-l01 directory and copy into it the following content:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hellofromnode-deployment
     labels:
       environment: dev
       app: hellofromnode
   spec:
     replicas: 1
     template:
       metadata:
         name: hellofromnode
         labels:
           app: hellofromnode
       spec:
         nodeSelector:
           "kubernetes.io/os": linux
         containers:
         - name: hellofromnode
           image: ACR_NAME.azurecr.io/hellofromnode:v1.0
           resources:
             limits:
               cpu: 1
               memory: 800M
           ports:
             - containerPort: 80
     selector:
       matchLabels:
         app: hellofromnode
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hellofromnode-service
   spec:
     type: LoadBalancer
     ports:
     - protocol: TCP
       port: 80
     selector:
       app: hellofromnode
   ```

   > **Note:** The deployment will create pods based on the Linux container image to the Linux node pool in the AKS cluster. In addition, the manifest includes a service, which will provide a load balanced access to the pods in the deployment via a public IP address on port 80.

1. Save the changes to the file and close it to return to the Bash prompt.
1. From the Bash session of Azure Cloud Shell, replace the **ACR_NAME** placeholder in the file aks-deployment-l01.yaml, by running the following commands:

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-l01.yaml
   ```

### Task 3: Create a Kubernetes manifest for deploying the Windows image
In this task, you will create a Kubernetes manifests for deploying the Windows image to the Windows node pool

1. In the Bash session of Azure Cloud Shell, create a directory that will host the deployment manifest for provisioning pods based on the Windows image and change to it from the current directory by running the following commands:

   ```bash
   mkdir ~/aks-w01
   cd ~/aks-w01
   ```

1. In the Bash session of Azure Cloud Shell, use the built-in editor to create a file named aks-deployment-w01.yaml in the aks-w01 directory and copy into it the following content:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hellofromdotnet-deployment
     labels:
       environment: dev
       app: hellofromdotnet
   spec:
     replicas: 1
     template:
       metadata:
         name: hellofromdotnet
         labels:
           app: hellofromdotnet
       spec:
         nodeSelector:
           "kubernetes.io/os": windows
         containers:
         - name: hellofromdotnet
           image: ACR_NAME.azurecr.io/hellofromdotnet:v1.0
           resources:
             limits:
               cpu: 1
               memory: 800M
           ports:
             - containerPort: 80
     selector:
       matchLabels:
         app: hellofromdotnet
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hellofromdotnet-service
   spec:
     type: LoadBalancer
     ports:
     - protocol: TCP
       port: 80
     selector:
       app: hellofromdotnet
   ```

   > **Note:** The deployment will create pods based on the Windows container image to the Windows node pool in the AKS cluster. In addition, the manifest includes a service, which will provide a load balanced access to the pods in the deployment via a public IP address on port 80.

1. Save the changes to the file and close it to return to the Bash prompt.
1. From the Bash session of Azure Cloud Shell, replace the **ACR_NAME** placeholder in the file aks-deployment-l01.yaml by running the following command:

   ```azurecli
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-w01.yaml
   ```

### Task 4: Perform AKS deployments by using YAML manifest files
In this task, you will deploy both container images to their respective namespaces and node pools in the target AKS cluster.

1. From the Bash session of Azure Cloud Shell, connect to the AKS cluster, by running the following commands:

   ```azurecli
   AKSRG='aks-01-RG'
   AKSNAME='aks-01'
   az aks get-credentials --resource-group $AKSRG --name $AKSNAME
   ```

1. From the Bash session of Azure Cloud Shell, verify that the connection was successful, by running the following command:

   ```kubectl
   kubectl get nodes
   ```

   > **Note:** The output of the command should include the listing of all AKS nodes (four in our case).

1. From the Bash session of Azure Cloud Shell, create the first deployment defined in the corresponding YAML manifest file in the **dev-node** namespace by running the following commands:

   ```kubectl
   cd ~/aks-l01
   kubectl apply -f aks-deployment-l01.yaml -n=dev-node
   ```

   > **Note:** Proceed to the next step without waiting for the deployment to complete. The provisioning of all resources might take a few minutes.

1. From the Bash session of Azure Cloud Shell, create the second deployment defined in the corresponding YAML manifest file by running the following command:

   ```kubectl
   cd ~/aks-w01
   kubectl apply -f aks-deployment-w01.yaml -n=dev-dotnet
   ```

   > **Note:** Proceed to the next step without waiting for the deployment to complete. The provisioning of all resources might take a few minutes.

