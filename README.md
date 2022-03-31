# ONLYOFFICE Docs-example for Kubernetes

This repository contains a set of files to deploy ONLYOFFICE Docs-example into a Kubernetes cluster.

## Introduction

- To install, you need a kubernetes cluster, for installation, read the [instructions](https://kubernetes.io/docs/setup/) 
- You will need kubectl installed, you can read the instructions at this [link](https://kubernetes.io/docs/tasks/tools/).
- You will also need Helm v3. For installation, read the [instructions]((https://helm.sh/docs/intro/install/)

## Installing dependencies  

### 1. Add Helm repositories

To install databases and nfs server you need to add repositories:

```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo add stable https://charts.helm.sh/stable
$ helm repo update
```
### 2. Install Persistent Storage

```bash
$ helm install nfs-server stable/nfs-server-provisioner \
  --set persistence.enabled=true \
  --set persistence.storageClass=PERSISTENT_STORAGE_CLASS \
  --set persistence.size=PERSISTENT_SIZE
```

- `PERSISTENT_STORAGE_CLASS` is a Persistent Storage Class available in your Kubernetes cluster.

  Persistent Storage Classes for different providers:
  - Amazon EKS: `gp2`
  - Digital Ocean: `do-block-storage`
  - IBM Cloud: Default `ibmc-file-bronze`. [More storage classes](https://cloud.ibm.com/docs/containers?topic=containers-file_storage)
  - Yandex Cloud: `yc-network-hdd` or `yc-network-ssd`. [More details](https://cloud.yandex.ru/docs/managed-kubernetes/operations/volumes/manage-storage-class)
  - minikube: `standard`

- `PERSISTENT_SIZE` is the total size of all Persistent Storages for the nfs Persistent Storage Class. You can express the size as a plain integer with one of these suffixes: `T`, `G`, `M`, `Ti`, `Gi`, `Mi`. For example: `9Gi`.

See more details about installing NFS Server Provisioner via Helm [here](https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner#nfs-server-provisioner).
Please note, installing an NFS server on bare metal kubernetes cluster requires a pre-created pv.

Configure a Persistent Volume Claim

Note: The default `nfs` Persistent Volume Claim is 8Gi. You can change it in the `values.yaml` file in the `persistence.storageClass` and `persistence.size` section. It should be less than `PERSISTENT_SIZE` at least by about 5%. It's recommended to use 8Gi or more for persistent storage for every 100 active users of ONLYOFFICE Docs.

### 3. Deploy ONLYOFFICE Docs-example

To deploy Docs-example with the release name `docs-example`:

```bash

$ helm install docs-example ./
```

The command deploys DocumentServer on the Kubernetes cluster in the default configuration. The Parameters section lists the parameters that can be configured during installation.

### 4. Uninstall ONLYOFFICE Docs-example

To uninstall/delete the `documentserver` deployment:

```bash
$ helm delete docs-example

```

### 5. Avalivable parameters

Below are the options available for configuration before deploying docs-example to a cluster: 

`persistence.storageClass` storage class name default: `nfs`
`persistence.size` storage volume size default: `8Gi`
`example.containerImage` example container image name default: `onlyoffice/docs-example:latest`
`example.imagePullPolicy` example container image pull policy default: `IfNotPresent`
`example.resources.requests.memory` memory request default: 
`example.resources.requests.cpu` cpu request default: 
`example.resources.limits.memory` memory limit default: 
`example.resources.limits.cpu` cpu limit defalut: 
`jwt.enabled` jwt enabling parameter default:
`jwt.secret` jwt secret default: 
`jwt.header` Defines the http header that will be used to send the JSON Web Token default: 
`jwt.inBody` Specifies the enabling the token validation in the request body to the ONLYOFFICE Docs default: 
`service.type` docs-example service type default: `LoadBalancer`
`service.port` docs-example service port default: `3000`
`ingress.enabled` installation of ingress service defaule: `false`
`ingress.host` Ingress hostname for the documentserver ingress default:	`""`
`ingress.ssl.enabled` installation ssl for ingress service default: `false`
`ingress.ssl.secret` secret name for ssl default: `tls`
