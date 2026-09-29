# acr

# Azure Container Registry (ACR) Foundations Project

## Overview

This project is a beginner-friendly introduction to **Azure Container Registry (ACR)**.

The goal is to learn how to create a private container registry in Azure, authenticate to it, push and pull Docker images, manage repositories and tags, and understand basic ACR access control.

---

## Project Objectives

By completing this project, I learned how to:

- Create an Azure Container Registry
- Authenticate to ACR using Azure CLI
- Pull a Docker image from Docker Hub
- Tag an image for Azure Container Registry
- Push images to ACR
- Pull images from ACR
- View repositories and image tags
- Inspect image manifests and digests
- Configure ACR repository permissions using Azure RBAC
- Troubleshoot common ACR authentication and authorization issues

---

## Technologies Used

- Microsoft Azure
- Azure Container Registry
- Azure CLI
- Docker
- Microsoft Entra ID
- Azure RBAC

---

## Architecture

```text
Docker Hub
    |
    v
Local Docker
    |
    | docker tag
    |
    v
Azure Container Registry
    |
    | docker pull
    |
    v
Local Docker / Future Azure Workloads
```

---

## Step 1 — Create Variables

```bash
RG="rg-acr-foundations"
LOCATION="canadacentral"
ACR_NAME="<unique-acr-name>"
```

Check whether the registry name is available:

```bash
az acr check-name \
  --name "$ACR_NAME"
```

---

## Step 2 — Create the Resource Group

```bash
az group create \
  --name "$RG" \
  --location "$LOCATION"
```

---

## Step 3 — Create Azure Container Registry

```bash
az acr create \
  --resource-group "$RG" \
  --name "$ACR_NAME" \
  --location "$LOCATION" \
  --sku Standard \
  --role-assignment-mode rbac-abac
```

---

## Step 4 — Get the ACR Login Server

```bash
LOGIN_SERVER=$(az acr show \
  --name "$ACR_NAME" \
  --resource-group "$RG" \
  --query loginServer \
  --output tsv)

echo "$LOGIN_SERVER"
```

Example:

```text
myregistry.azurecr.io
```

---

## Step 5 — Get the ACR Resource ID

```bash
ACR_ID=$(az acr show \
  --name "$ACR_NAME" \
  --resource-group "$RG" \
  --query id \
  --output tsv)

echo "$ACR_ID"
```

---

## Step 6 — Get the Current User ID

```bash
USER_ID=$(az ad signed-in-user show \
  --query id \
  --output tsv)

echo "$USER_ID"
```

---

## Step 7 — Assign Repository Permissions

Give the signed-in user permission to push and manage images:

```bash
az role assignment create \
  --assignee-object-id "$USER_ID" \
  --assignee-principal-type User \
  --role "Container Registry Repository Writer" \
  --scope "$ACR_ID"
```

Allow the user to list repositories:

```bash
az role assignment create \
  --assignee-object-id "$USER_ID" \
  --assignee-principal-type User \
  --role "Container Registry Repository Catalog Lister" \
  --scope "$ACR_ID"
```

---

## Step 8 — Login to ACR

```bash
az acr login \
  --name "$ACR_NAME"
```

Expected result:

```text
Login Succeeded
```

---

## Step 9 — Pull a Docker Image

```bash
docker pull nginx:alpine
```

Verify:

```bash
docker images