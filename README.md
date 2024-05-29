---
layout: page
title: IN2 DOME GitOps
version: 0.0.1
date: 2024-05-29
editor:
    name: Oriol Canadés
    email: oriol.canades@in2.es 
---

<h1>IN2 DOME GitOps</h1>

<h2>Table of Contents</h2>

# Prerequisites
- ionosctl-cli
- jq
- kubectl
- kubeseal

# Infrastructure Setup (DataCenter, Kubernetes Cluster, Node-Pools)

## 1. Login into IONOS using the ionosctl-cli

> NOTE: to get the token, you will need to access the [IONOS Cloud Panel](https://dcd.ionos.com/) and create a new API token: Management > Token Management > Generate Token.

```bash
ionosctl login -t <token>
```

## 2. Create a DataCenter in IONOS

```bash
export IN2_DOME_DEV_DATACENTER_ID=$(ionosctl datacenter create --name in2-ssi-dev -o json | jq -r '.items[0].id')
watch ionosctl datacenter get -i $IN2_DOME_DEV_DATACENTER_ID
```

## 3. Create a Kubernetes cluster

```bash
export IN2_DOME_DEV_K8S_CLUSTER_ID=$(ionosctl k8s cluster create --name in2-ssi-dev-k8s -o json | jq -r '.items[0].id')
watch ionosctl k8s cluster get -i $IN2_DOME_DEV_K8S_CLUSTER_ID
```

## 4. Create a Node-Pool

```bash
export IN2_DOME_DEV_K8S_DEFAULT_NODE_POOL_ID=$(ionosctl k8s nodepool create --cluster-id $IN2_DOME_DEV_K8S_CLUSTER_ID --name default-pool --node-count 2 --ram 8192 --storage-size 10 --datacenter-id $IN2_DOME_DEV_DATACENTER_ID --cpu-family "INTEL_SKYLAKE"  -o json | jq -r '.items[0].id')
watch ionosctl k8s nodepool get --nodepool-id $IN2_DOME_K8S_DEFAULT_NODEPOOL_ID --cluster-id $IN2_DOME_DEV_K8S_CLUSTER_ID
```

## 5. Create a Node-Pool for Ingress

```bash
ionosctl k8s nodepool create --cluster-id $IN2_DOME_DEV_K8S_CLUSTER_ID \    --name ingress --node-count 1 --datacenter-id $IN2_DOME_DEV_DATACENTER_ID --cpu-family "INTEL_SKYLAKE" --labels nodepool=ingress
```

## 6. Get the kubeconfig file and set the KUBECONFIG variable

```bash
ionosctl k8s kubeconfig get --cluster-id $IN2_DOME_DEV_K8S_CLUSTER_ID > in2-dome-dev-k8s-config.json
export KUBECONFIG=$(pwd)/in2-dome-dev-k8s-config.json
```

## 7. Validate the contexts that are reachable

```bash
kubectl config get-contexts
```

## 8. Access to the context

```bash
kubectl config use-context cluster-admin@in2-dome-dev-k8s
```

# GitOps Setup

## 1. Create ArgoCD namespace

```bash
kubectl create namespace argocd
```

## 2 Deploy ArgoCD with Extensions

```bash
kubectl apply -k ./extension/ -n argocd
```

## 3. Deploying Namespaces Application

```bash
kubectl apply -f applications_dev/namespaces.yaml -n argocd
```

## 4. Deploying Sealed-Secrets Application

> NOTE: This environment is for dev purposes, we do not need to encrypt the secrets.

```bash
kubectl apply -f applications_dev/sealed-secrets.yaml -n argocd
```

## 5. Deploying Ingress Application

```bash
kubectl apply -f applications_dev/ingress.yaml -n argocd
```

> NOTE: You can get all the applications deployed in the argocd namespace by running the following command:

```bash
kubectl get applications -n argocd
```

# ArgoCD

## 1. Install the ArgoCD CLI

```bash
brew install argocd
```

## 2. Access to the ArgoCD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

> NOTE: Open your browser and go to [http://localhost:8080](http://localhost:8080) and login with the default credentials: `admin`/`admin`.

## 3. Get the ArgoCD password

```bash
argocd admin initial-password -n argocd
```
