---
layout: post
title: "Kubernetes Ephemeral Containers for Debugging and Troubleshooting"
date: 2025-08-20 21:00:00 -07:00
categories: kubernetes
tags:
- kubernetes
- containers
- aks
---

I recently had to run a vendor's shell script to patch a solution running in Azure Kubernetes Services. In order to do this quickly and efficiently, I used an ephemeral container in the AKS cluster to run the shell script.

## Basic Steps

1. PIM up to Owner on Azure subscription where Azure Kubernetes Services cluster exists
2. Open PowerShell 7 session with az cli and kubectl available
3. Log into azure
4. Run ephemeral (temporary) pod with kubectl in the AKS cluster
5. Configure all required things in the ephemeral pod
6. Run vendor's provided shell commands
7. Exit / Remove ephemeral pod

## Get logged in and run pod with azure-cli

```powershell
# Log in to azure - make sure to use account that was pimmed up on
az login
# Set azure account context to desired subscription
az account set --subscription 'subscription name'
# use az account show to confirm if desired

# Get credentials for Azure Kubernetes Services cluster
az aks get-credentials --resource-group <rg-name> --name <aks-cluster-name>

# Start an ephemeral container based on the microsoft azure cli
# Ref: https://learn.microsoft.com/en-us/cli/azure/run-azure-cli-docker?view=azure-cli-latest
kubectl run tmp-shell -i --tty --image mcr.microsoft.com/azure-cli:azurelinux3.0
```

## Configure the ephemeral pod

`kubectl run ...` should result in having a root shell prompt, at root of the container's file system

```bash
# it will dump you to root, change to root's home
cd ~

# log into azure *inside the pod*
az login
# Set context to Azure subscription *inside the pod*
az account set --subscription 'subscription name'
# Get aks cluster credentials *inside the pod*
az aks get-credentials --resource-group <rg-name> --name <aks-cluster-name>

# install kubectl because it is not included in microsoft's azure cli container
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl

# set kubectl to executable
chmod +x kubectl

# make a directory structure for kubectl
mkdir .local
mkdir .local/bin

# put kubectl in directory
mv kubectl .local/bin
# add directory to path
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
# reload bashrc
source ~/.bashrc
# check that it works
kubectl

# get context
kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO                                       NAMESPACE
*         aks-cluster-name aks-cluster-name clusterUser_rg-name_aks-cluster-name

# step 1 of vendor provided commands (curl to get the shell script) downloads the script
curl -JLO https://vendor.domain.com/download/guid/ 

# step 2 make the downloaded shell script executable
chmod +x vendor_migration-v3-1.sh

# step 3 execute
./vendor_migration-v3-1.sh

# exit ephemeral pod
exit
```

## Clean up ephemeral pod

After completed, delete the ephemeral pod

```powershell
kubectl delete pod tmp-shell
```
