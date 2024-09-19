# AKS ArgoCD setup guide

This repo contains instructions on how to setup [ArgoCD](https://argo-cd.readthedocs.io) with AKS

## Prerequisites
* [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/)
* [Helm](https://helm.sh/docs/intro/install/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)


## Create AKS cluster

1. Create a resource group using the `az group create` command.

    ```bash
    RG=aks-argocd
    REGION=eastus2
    az login
    az group create --name $RG --location $REGION
    ```

1. Create AKS cluster:
    ```bash
    CLUSTER=aks-argocd-cluster
    az aks create --resource-group $RG --name $CLUSTER
    ```
    
1. Log into AKS cluster:
    ```bash
    az aks get-credentials --resource-group $   RG --name $CLUSTER --overwrite-existing
    ```

## Install ArgoCD

1. Use the following `kubectl` commands to install ArgoCD on the cluster:
    ```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

1. Change the `argocd-server` service to `LoadBalancer` so that it can be reached via public IP:

    ```bash
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
    ```

1. Get the `EXTERNAL-IP` Address for the `argocd-server` service:
    ```bash
    kubectl get svc argocd-server -n argocd
    ```

1. Get the password to log into ArgoCD console:
    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```

1. Navigate to the External IP in a browser above and provide `admin` as the username and the password value above to log into the ArgoCD console.

## Deploy your first ArgoCD application

1. Export project name to use:

    ```bash
    NAMESPACE=opa-gitops
    ```

1. Create project

    ```bash
    kubectl create namespace $NAMESPACE
    ```

1. Change to `infra/gitops` directory:

    ```bash
    cd infra/gitops
    ```

1. Install `gitops-app-project` chart:

    ```bash
    helm upgrade -i opa-gitops gitops-app-project -f ../helm/values/dev/values.yaml \
    --set "app.namespace=$NAMESPACE"
    ```