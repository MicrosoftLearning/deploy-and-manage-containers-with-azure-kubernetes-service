---
Guided Exercise:
  title: Guided Exercise Deploy Applications to AKS
---
# Guided Exercise - Deploy applications to Azure Kubernetes Service (AKS)

## Objectives

This guided exercise consist of the following activities:

+ Exercise 1: Provision Azure Container Registry (ACR) and Azure Kubernetes Service (AKS)
+ Exercise 2: Build a Linux and Windows container images and store them in ACR
+ Exercise 3: Deploy container images to AKS 
+ Exercise 4: Review the deployment and deprovision all resources

## Exercise 1: Provision Azure Container Registry (ACR) and Azure Kubernetes Service (AKS)
In this exercise, you will create an Azure Container registry and an AKS cluster.


>**Note**: To complete this exercise you will need an [Azure subscription](https://azure.microsoft.com/free/).
> For any properties that are not specified, use the default value.


### Task 1: Create an Azure Container registry
In this task, you will create an Azure Container registry

1. From your computer, open a web browser window and navigate to the Azure portal at https://portal.azure.com.
1. When prompted, sign in by using a user account that has the Owner role in the Azure subscription you will be using in this exercise. 
1. Sign in to the Azure portal. 
1. In the Azure portal, in the **Search** text box, search for and select **Container registries**.
1. On the **Container registries** page, select **+ Create** and specify the following settings:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you will be using in this exercise|
    |Resource Group|The name of a new resource group **acr-01-RG**|
    |Registry name|Any valid, globally unique name consisting of between 5 and 50 alphanumeric characters|
    |Region|Any Azure region in which you can create an Azure Container registry and an AKS cluster|
    |Availability zones|**None**|
    |SKU|**Basic**|

1. On the **Container registries** page, select **Review + create** and, on the **Review + create** tab, select **Create**.

   > **Note:** Proceed to the next exercise without waiting for the provisioning of the Azure Container registry to complete.

### Task 2: Create an Azure virtual network and an AKS cluster
In this task, you will create an Azure virtual network and deploy an AKS cluster including a Windows node pool into that virtual network.

> **Note:** While you could create a virtual network when provisioning an AKS cluster, this is not a typical scenario. More importantly, deploying AKS clusters into an existing virtual network requires some additional considerations that you should be familiar with.

1. In the Azure portal, in the **Search** text box, search for and select **Virtual networks**.
1. On the **Virtual networks** page, select **+ Create** and then, on the **Basics** tab of the **Create virtual network** page, specify the following settings:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you selected in the first exercise|
   |Resource Group|The name of a new resource group **aks-01-RG**|
   |Virtual network name|**vnet-01**|
   |Region|The same Azure region you selected in the first exercise of this exercise|

1. On the **Basics** tab of the **Create virtual network** page, select **Next**.
1. On the **Security** tab of the **Create virtual network** page, accept the default settings and select **Next**.
1. On the **IP addresses** tab of the **Create virtual network** page, ensure that the IP address space is set to **10.0.0.0/16**, delete the **default** subnet, select **Review + create** and, on the **Review + create** tab, select **Create**.

   > **Note:** Creating a virtual network should take only a few seconds, so you should be able to proceed directly to the next step.

1. In the Azure portal, in the **Search** text box, search for and select **Kubernetes services**.
1. On the **Kubernetes services** page, select **+ Create**, in the dropdown list, select **Create a Kubernetes cluster**, and then, on the **Basics** tab of the **Create Kubernetes cluster** page, specify the following settings and then select **Next**:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you selected in the first exercise of this lab|
   |Resource Group|**aks-01-RG**|
   |Cluster preset configuration|**Dev/Test**|
   |Kubernetes cluster name|**aks-01**|
   |Region|The same Azure region you selected in the first exercise of this lab|
   |Availability zones|**None**|
   |AKS pricing tier|**Free**|
   |Kubernetes version|Accept the default value|
   |Automatic upgrade|Disabled|
   |Node security channel type|**None**|
   |Authentication and Authorization|**Local accounts with Kubernetes RBAC**|

1. On the **Node pools** tab of the **Create Kubernetes cluster** page, perform the following tasks:

   - In the **Node pools** section, select the **agentpool** link.
   - On the **Update node pool** page, in the **Node size** section, select the **Choose a size** link.
   - On the **Select a VM size** page, in the list of VM sizes, select **B4ms** and then click **Select**.
   - Back on the **Update node pool** page, set **Scale method** to **Manual** and **Node count** to **2**.
   - On the **Update node pool** page, select **Update**.

   > **Note:** You might need to increase vCPU quotas or change the VM SKU to accommodate the node size and node count values. For information regarding the procedure to increase vCPU quotas, refer to the Microsoft Learn article [Increase VM-family vCPU quotas](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests).

   > **Note:** You will add a Windows node pool to the cluster. This required changing the network configuration to **Azure CNI** from the default **Kubenet**. The Kubenet network configuration does not support Windows node pools.

1. Back on the **Node pools** tab of the **Create Kubernetes cluster** page, select **Next**.
1. On the **Networking** tab of the **Create Kubernetes cluster** page, select the **Azure CNI** option,, select the **Bring your own virtual network** checkbox,  in the **Virtual network** dropdown list, select **vnet-01**, and below the **Cluster subnet** textbox, select **Manage subnet configuration**.
1. On the **vnet-01 \| Subnets** page, select **+ Subnet**.
1. On the **Add subnets** page, specify the following settings and select **Save**:

   |Setting|Value|
   |---|---|
   |Name|**aks-subnet**|
   |Subnet address range|**10.0.0.0/20**|

1. Back on the **vnet-01 \| Subnets** page, in the breadcrumb trail in the top-left part of the page, select **Create Kubernetes cluster**. 
1. Back on the **Networking** tab of the **Create Kubernetes cluster** page, specify the following settings:

   |Setting|Value|
   |---|---|
   |Virtual network|**vnet-01**|
   |Cluster subnet|**aks-subnet (10.0.0.0/20)**|
   |Kubernetes service address range|**172.16.0.0/22**|
   |Kubernetes DNS service IP address|**172.16.3.254**|
   |DNS name prefix|**aks-01-dns**|
   |Network policy|**None**|

1. On the **Networking** tab of the **Create Kubernetes cluster** page, select **Previous**.
1. Back on the **Node pools** tab of the **Create Kubernetes cluster** page, select **+ Add node pool**.
1. On the **Add node pool** page, specify the following settings:

   |Setting|Value|
   |---|---|
   |Node pool name|**w1pool**|
   |Mode|**User**|
   |OS type|**Windows 2022**|
   |Availability zone|**None**|
   |Enable Azure spot instances|Disabled|
   |Node size|**B4ms**|
   |Scale method|**Manual**|
   |Node count|**2**|
   |Max pods per node|**30**|
   |Enable public IP per node|Disabled|

   > **Note:** Here as well you might need to increase vCPU quotas or change the VM SKU to accommodate the node size and node count values.

1. On the **Add node pool** page, select **Add**.
1. Back on the **Node pools** tab of the **Create Kubernetes cluster** page, select **Next**.
1. On the **Networking** tab of the **Create Kubernetes cluster** page, select **Next**.
1. On the **Integration** tab of the **Create Kubernetes cluster** page, in the **Container registry** dropdown list, select the entry representing the Azure Container registry you created in the previous exercise, disable the **Enable recommended alert rules** checkbox, ensure that **Azure Policy** option is disabled, and select **Next**.
1. On the **Monitoring** tab of the **Create Kubernetes cluster** page, de-select the **Enable Prometheus metrics** checkbox and then select **Review + create**.
1. On the **Review + create** tab of the **Create Kubernetes cluster** page, select **Create**.

   > **Note:** Proceed to the next exercise without waiting for the provisioning of AKS cluster to complete. The provisioning process might take about 5 minutes.

## Exercise 2: Build a Linux and Windows container images and store them in ACR
In this exercise, you will build a Linux- and Windows-based Docker images and pushed them into the Azure Container registry you created earlier in this exercise .

### Task 1: Build a Linux container image and store it in ACR
In this task, you will use a ACR task to build a Linux container image and automatically push it into the ACR.

1. In the Azure portal, select the **Cloud Shell** icon.
1. If prompted to select either **Bash** or **PowerShell**, select **Bash**. 
1. If prompted, select **Create storage**, and wait until the Azure Cloud Shell pane is displayed. 
1. Ensure that **Bash** appears in the drop-down menu in the upper-left corner of the Cloud Shell pane.
1. From the Bash session within Cloud Shell, create a directory that will host the Dockerfile for the Linux image and change to it from the current directory by running the following commands:

   ```bash
   mkdir ~/image-l01
   cd ~/image-l01
   ```

1. In the Bash session of Azure Cloud Shell, use the built-in editor to create a file named server.js in the image-l01 directory and copy into it the following content:

   ```js
   const http = require('http')
   const port = 80
   const server = http.createServer((request, response) => {
     response.writeHead(200, {'Content-Type': 'text/plain'})
     response.write('Hello World from Node\n')
     response.end('Version: ' + process.env.NODE_VERSION + '\n')
   })
   server.listen(port)
   console.log(`Server running at http://localhost: ${port}`)
   ```

   > **Note:** Once running, the resulting Node.js code will display the message **Hello World from Node**.

1. In the Bash session of Azure Cloud Shell, use the built-in editor again to create a file named package.json in the image-l01 directory and copy into it the following content:

   ```json
   {
     "name": "helloworld",
     "version": "1.0.0",
     "description": "Sample app for ACR Build",
     "main": "server.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       "start": "node server.js"
     },
     "license": "MIT"
   }
   ```

1. Save the changes to the file and close it to return to the Bash prompt.
1. In the Bash session of Azure Cloud Shell, use the built-in editor again to create a file named Dockerfile in the image-l01 directory and copy into it the following content:

   ```Dockerfile
   FROM node:20.2-alpine
   COPY . /src
   RUN cd /src && npm install
   EXPOSE 80
   CMD ["node", "/src/server.js"]
   ```

1. Save the changes to the file and close it to return to the Bash prompt.
1. From the Bash session of Azure Cloud Shell, identify the name of your Azure Container registry you created earlier in this exercise and store it in a variable named **$ACRNAME** by running the following commands:

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   ```

1. From the Bash session of Azure Cloud Shell, create a Docker image based on the Dockerfile stored in the current directory and push it automatically to the Azure Container registry which name is stored in the **$ACRNAME** variable by running the following command (make sure to include the trailing period):

   ```azurecli
   az acr build --registry $ACR_NAME --image hellofromnode:v1.0 .
   ```

   > **Note:** Track the progress of the build and make sure it completes successfully. This should take less than 1 minute.

### Task 2: Build a Windows container image and store it in ACR
In this task, you will use a ACR task to build a Windows container image and automatically push it into the ACR.

1. From the Bash session within Cloud Shell, create a directory that will host the Dockerfile for the Windows image and change to it from the current directory by running the following commands:

   ```bash
   mkdir ~/image-w01
   cd ~/image-w01
   ```

1. From the Bash session of Azure Cloud Shell, clone a public GitHub repo hosting the files you'll use to build the Windows image and switch the current directory to the cloned repo by running the following commands:

   ```git
   git clone https://github.com/Azure-Samples/dotnetcore-docs-hello-world.git
   cd ~/image-w01/dotnetcore-docs-hello-world
   ```

   > **Note:** The repo contains the source code for a .NET 7 web app which displays the message **Hello World from .NET 7**.

1. From the Bash session of Azure Cloud Shell, create a Docker image based on the Dockerfile stored in the current directory and push it automatically to the Azure Container registry which name is stored in the **$ACRNAME** variable by running the following command (make sure to include the trailing period):

   ```azurecli
   az acr build --registry $ACR_NAME --image hellofromdotnet:v1.0 --platform windows --file Dockerfile.windows .
   ```

   > **Note:** The name of the Dockerfile defining the Windows image build is **Dockerfile.windows**.

   > **Note:** Track the progress of the build and make sure it completes successfully. This should take less than 3 minutes.

1. Close the Azure Cloud Shell pane.
1. In the Azure portal, navigate to the **Container registries** page and select the entry representing the Container registry to which you pushed both images.
1. On the Container registry page, in the vertical hub menu, select **Repositories** and verify that **hellofromnode** and **hellofromdotnet** appear in the list of repositories.

## Exercise 3: Deploy container images to AKS 
In this exercise, you will deploy two container images you created earlier in this exercise to the AKS cluster.

> **Note:** Before you proceed with this exercise, make sure that the provisioning of the AKS cluster has successfully completed.

### Task 1: Create custom AKS namespaces
In this task, you will create two namespaces on the AKS cluster you created earlier in this exercise .

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

# Exercise 4: Review the deployment and deprovision all resources
In this exercise, you will review the results of the deployments and deprovision all resources.

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
In this task, you will delete all resources provisioned in this exercise.

1. From the Bash session of Azure Cloud Shell, display the listing of resources in the two resource groups provisioned in this exerciseby running the following commands:

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
