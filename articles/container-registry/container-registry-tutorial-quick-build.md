---
title: Tutorial - Build container images in the cloud with Azure Container Registry Build
description: In this tutorial, you learn how to build a Docker container image in Azure with Azure Container Registry Build (ACR Build), then deploy it to Azure Container Instances.
services: container-registry
author: mmacy
manager: jeconnoc

ms.service: container-registry
ms.topic: tutorial
ms.date: 04/23/2018
ms.author: marsma
ms.custom: mvc
# Customer intent: As a developer or devops engineer, I want to quickly build
# container images in Azure, without having to install dependencies like Docker
# Engine, so that I can simplify my inner-loop development pipeline.
---

# Tutorial: Build container images in the cloud with Azure Container Registry Build

**ACR Build** is a suite of features within Azure Container Registry that provides streamlined and efficient Docker container image builds in Azure. In this article, you learn how to use the *Quick Build* feature of ACR Build. Quick Build extends your development "inner loop" to the cloud, providing you with build success validation and automatic pushing of successfully built images to your container registry. Your images are built natively in the cloud, close to your registry, enabling faster deployment.

All your Dockerfile expertise is directly transferrable to ACR Build. You don't have to change your Dockerfiles to build in the cloud with ACR Build, just the command you run.

In this tutorial, part one of a series:

> [!div class="checklist"]
> * Get ACR Build
> * Get the sample application source code
> * Build a container image in Azure
> * Deploy a container to Azure Container Instances

In subsequent tutorials, you learn to use ACR Build's build tasks for automated container image builds on code commit and base image update.

> [!IMPORTANT]
> ACR Build is in currently in preview, and is supported only by Azure container registries in the **EastUS** region. Previews are made available to you on the condition that you agree to the [supplemental terms of use][terms-of-use]. Some aspects of this feature may change prior to general availability (GA).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

If you'd like to use the Azure CLI locally, you must have Azure CLI version 2.0.31 or later installed. Run `az --version` to find the version. If you need to install or upgrade the CLI, see [Install Azure CLI 2.0][azure-cli].

## Prerequisites

### Get ACR Build

While ACR Build is in preview, you must manually install the **acrbuildext** Azure CLI extension using the following steps.

Uninstall any previous version of the extension:

```azurecli-interactive
az extension remove -n acrbuildext
```

Install the current version of the extension:

```azurecli-interactive
az extension add --source https://acrbuild.blob.core.windows.net/cli/acrbuildext-0.0.4-py2.py3-none-any.whl -y
```

Execute `az acr build --help` to verify successful installation:

```console
$ az acr build --help
This command is from the following extension: acrbuildex
tCommand
    az acr build: Queues a new build based on the specified parameters
.Arguments
    --context -c  [Required]: The local source code directory path (eg, './src') or the url to a git
                              repository (eg, 'https://github.com/docker/rootfs.git') or a remote
                              tarball (eg, 'http://server/context.tar.gz').
    --registry -r [Required]: The name of the container registry. You can configure the default
                              registry name using `az configure --defaults acr=<registry name>`.
[...]
```

### GitHub account

Create an account on https://github.com if you don't already have one. This tutorial series uses a GitHub repository to demonstrate automated image builds in ACR Build.

### Fork sample repository

Next, use the GitHub UI to fork the sample repository into your GitHub account. In this tutorial, you build a container image from the source in the repo, and in the next tutorial, you push a commit to your fork of the repo to kick off an automated build.

Fork this repository: https://github.com/Azure-Samples/acr-build-helloworld-node

![Screenshot of the Fork button (highlighted) in GitHub][quick-build-01-fork]

### Clone your fork

Once you've forked the repo, clone your fork and enter the directory containing your local clone.

Clone the repo with `git`, replace **\<your-github-username\>** with your GitHub username:

```azurecli-interactive
git clone https://github.com/<your-github-username>/acr-build-helloworld-node
```

Enter the directory containing the source code:

```azurecli-interactive
cd acr-build-helloworld-node
```

## Build in Azure with ACR Build

Now that you've pulled the source code down to your machine, follow these steps to create a container registry and build the container image with ACR Build.

The following example commands create an Azure container registry named **mycontainerregistry**. Because this registry name might already be taken, replace **mycontainerregistry** with a unique name for your registry. The registry name must be unique within Azure and contain 5-50 alphanumeric characters. You're also welcome to change the resource group name defined in `RES_GROUP`.

> [!NOTE]
> ACR Build is currently supported only by registries in **EastUS**. Do not change the location of the resource group.

Create a resource group and an Azure container registry:

```azurecli-interactive
ACR_NAME=mycontainerregistry # Registry name - must be *unique* within Azure
RES_GROUP=$ACR_NAME # Resource Group name

az group create --resource-group $RES_GROUP --location eastus
az acr create --resource-group $RES_GROUP --name $ACR_NAME --sku Standard
```

Use ACR Build to build a container image from the sample code:

```azurecli-interactive
az acr build --registry $ACR_NAME --image helloacrbuild:v1 --context .
```

Output from the [az acr build][az-acr-build] command is similar to the following. You can see the upload of the source code (the "context") to Azure, and the details of the `docker build` operation that ACR Build runs in the cloud. Because ACR Build uses `docker build` to build your images, no changes to your Dockerfiles are required to start using ACR Build immediately.

```console
$ az acr build --registry $ACR_NAME --image helloacrbuild:v1 --context .
Sending build context (41.042 KiB) to ACR
Queued a build with ID: eastus-1
Sending build context to Docker daemon  191.5kB
Step 1/5 : FROM node:9-alpine
9-alpine: Pulling from library/node
605ce1bd3f31: Pulling fs layer
f10758dcda1f: Pulling fs layer
4cbe43d669e5: Pulling fs layer

[...]

Removing intermediate container 2dbac02fb0e6
 ---> 670bbc77128d
Step 4/5 : EXPOSE 80
 ---> Running in 1d09ee337a47
Removing intermediate container 1d09ee337a47
 ---> f0739d333913
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in 1f019c4e4b24
Removing intermediate container 1f019c4e4b24
fbd7c8b9c17e: Preparing
26b0c207c4a9: Preparing
917e7cdebc8b: Preparing
9dfa40a0da3b: Preparing
7d7224b439b3: Pushed
9dfa40a0da3b: Pushed
fbd7c8b9c17e: Pushed
26b0c207c4a9: Pushed
917e7cdebc8b: Pushed
v1: digest: sha256:60d78f0a336a387ba93f04ecf22538d01bca985a277ac77d3813ce360aba0cb1 size: 1367
time="2018-04-18T18:28:29Z" level=info msg="Running command docker inspect --format \"{{json .RepoDigests}}\" mycontainerregistry.azurecr.io/helloacrbuild:v1"
"["mycontainerregistry.azurecr.io/helloacrbuild@sha256:60d78f0a336a387ba93f04ecf22538d01bca985a277ac77d3813ce360aba0cb1"]"
time="2018-04-18T18:28:30Z" level=info msg="Running command docker inspect --format \"{{json .RepoDigests}}\" node:9-alpine"
"["node@sha256:5149aec8f508d48998e6230cdc8e6832cba192088b442c8ef7e23df3c6892cd3"]"
ACR Builder discovered the following dependencies:
[{"image":{"registry":"mycontainerregistry.azurecr.io","repository":"helloacrbuild","tag":"v1","digest":"sha256:60d78f0a336a387ba93f04ecf22538d01bca985a277ac77d3813ce360aba0cb1"},"runtime-dependency":{"registry":"registry.hub.docker.com","repository":"node","tag":"9-alpine","digest":"sha256:5149aec8f508d48998e6230cdc8e6832cba192088b442c8ef7e23df3c6892cd3"},"buildtime-dependency":null}]
Build complete
Build ID: eastus-1 was successful after 38.116951381s
```

Near the end of the output, ACR Build displays the dependencies it's discovered for your image. This enables ACR Build to automate image builds on base image updates, such as when a base image is updated with OS or framework patches. You learn about ACR Build's support for base image updates later in this tutorial series.

## Deploy to Azure Container Instances

ACR Build automatically pushes successfully built images to your registry by default, allowing you to deploy them from your registry immediately.

In this section, you create an Azure Key Vault and service principal, then deploy the container to Azure Container Instances (ACI) using the service principal's credentials.

### Configure registry authentication

All production scenarios should use [service principals][service-principal-auth] to access an Azure container registry. Service principals allow you to provide role-based access control to your container images. For example, you can configure a service principal with pull-only access to a registry.

#### Create key vault

If you don't already have a vault in [Azure Key Vault](/azure/key-vault/), create one with the Azure CLI using the following commands.

```azurecli-interactive
AKV_NAME=$ACR_NAME-vault

az keyvault create --resource-group $RES_GROUP --name $AKV_NAME
```

#### Create service principal and store credentials

You now need to create a service principal and store its credentials in your key vault.

Use the [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] command to create the service principal, and [az keyvault secret set][az-keyvault-secret-set] to store the service principal's **password** in the vault:

```azurecli-interactive
# Create service principal, store its password in AKV (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name $ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role reader \
                --query password \
                --output tsv)
```

The `--role` argument in the preceding command configures the service principal with the *reader* role, which grants it pull-only access to the registry. To grant both push and pull access, change the `--role` argument to *contributor*.

Next, store the service principal's *appId* in the vault, which is the **username** you pass to Azure Container Registry for authentication:

```azurecli-interactive
# Store service principal ID in AKV (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

You've created an Azure Key Vault and stored two secrets in it:

* `$ACR_NAME-pull-usr`: The service principal ID, for use as the container registry **username**.
* `$ACR_NAME-pull-pwd`: The service principal password, for use as the container registry **password**.

You can now reference these secrets by name when you or your applications and services pull images from the registry.

### Deploy container with Azure CLI

Now that the service principal credentials are stored as Azure Key Vault secrets, your applications and services can use them to access your private registry.

Execute the following [az container create][az-container-create] command to deploy a container instance. The command uses the service principal's credentials stored in Azure Key Vault to authenticate to your container registry.

```azurecli-interactive
az container create \
    --resource-group $RES_GROUP \
    --name acr-build \
    --image $ACR_NAME.azurecr.io/helloacrbuild:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label acr-build-$ACR_NAME \
    --query "{FQDN:ipAddress.fqdn}" \
    --output table
```

The `--dns-name-label` value must be unique within Azure, so the preceding command appends your container registry's name to the container's DNS name label. The output from the command displays the container's fully qualified domain name (FQDN), for example:

```console
$ az container create \
>     --resource-group $RES_GROUP \
>     --name acr-build \
>     --image $ACR_NAME.azurecr.io/helloacrbuild:v1 \
>     --registry-login-server $ACR_NAME.azurecr.io \
>     --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
>     --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
>     --dns-name-label acr-build-$ACR_NAME \
>     --query "{FQDN:ipAddress.fqdn}" \
>     --output table
FQDN
-------------------------------------------
acr-build-1175.eastus.azurecontainer.io
```

Take note of the container's FQDN, you'll use it in the next section.

### Verify deployment

To watch the startup process of the container, use the [az container attach][az-container-attach] command:

```azurecli-interactive
az container attach --resource-group $RES_GROUP --name acr-build
```

The `az container attach` output first displays the container's status as it pulls its image and starts, then binds your local console's STDOUT and STDERR to that of the container's.

```console
$ az container attach --resource-group $RES_GROUP --name acr-build
Container 'acr-build' is in state 'Waiting'...
Container 'acr-build' is in state 'Running'...
(count: 1) (last timestamp: 2018-04-03 19:45:37+00:00) pulling image "mycontainerregistry.azurecr.io/helloacrbuild:v1"
(count: 1) (last timestamp: 2018-04-03 19:45:44+00:00) Successfully pulled image "mycontainerregistry.azurecr.io/helloacrbuild:v1"
(count: 1) (last timestamp: 2018-04-03 19:45:44+00:00) Created container with id 094ab4da40138b36ca15fc2ad5cac351c358a7540a32e22b52f78e96a4cb3413
(count: 1) (last timestamp: 2018-04-03 19:45:44+00:00) Started container with id 094ab4da40138b36ca15fc2ad5cac351c358a7540a32e22b52f78e96a4cb3413

Start streaming logs:
Server running at http://localhost:80
```

When `Server running at http://localhost:80` appears, navigate to the container's FQDN in your brower to see the running application:

![Screenshot of sample application rendered in browser][quick-build-02-browser]

To detach your console from the container, hit `Control+C`.

## Clean up resources

Stop the container instance with the [az container delete][az-container-delete] command:

```azurecli-interactive
az container delete --resource-group $RES_GROUP --name acr-build
```

To remove *all* resources you've created in this tutorial, including the container registry, key vault, and service principal, issue the following commands. These resources are used in the [next tutorial](container-registry-tutorial-build-task.md) in the series, however, so you might want to keep them if you move on directly to the next tutorial.

```azurecli-interactive
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## Next steps

Now that you've tested your inner loop with a quick build, configure a **build task** to trigger container images builds when you commit source code to a Git repository:

> [!div class="nextstepaction"]
> [Trigger automatic builds with build tasks](container-registry-tutorial-build-task.md)

<!-- LINKS - External -->
[sample-archive]: https://github.com/Azure-Samples/acr-build-helloworld-node/archive/master.zip
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-create]: /cli/azure/container#az-container-create
[az-container-delete]: /cli/azure/container#az-container-delete
[az-keyvault-create]: /cli/azure/keyvault/secret#az-keyvault-create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[service-principal-auth]: container-registry-auth-service-principal.md

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
