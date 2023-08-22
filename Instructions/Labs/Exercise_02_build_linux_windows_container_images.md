---
Exercise:
  title: 'Exercise: Build Linux and Windows container images and store them in Azure Container Registry'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Exercise 2 - Deploy applications to Azure Kubernetes Service (AKS)

## Objectives

This guided project consist of the following exercises:

+ Exercise 1: Provision Azure Container Registry (ACR) and Azure Kubernetes Service (AKS).
+ **Exercise 2: Build a Linux and Windows container images and store them in Azure Container Registry.**
+ Exercise 3: Deploy container images to Azure Kubernetes Service.
+ Exercise 4: Review the deployment and deprovision all resources.

In this exercise you build a Linux and Windows container images and store them in Azure Container Registry.

## Exercise 2: Build a Linux and Windows container images and store them in ACR
In this exercise, you will build a Linux- and Windows-based Docker images and pushed them into the Azure Container registry you created earlier in this lab.


>**Note**: To complete this exercise you will need an [Azure subscription](https://azure.microsoft.com/free/).
> For any properties that are not specified, use the default value.


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
