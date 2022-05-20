# ONLYOFFICE Docs-example for Kubernetes

[![Release Chart on S3 repo](https://github.com/ONLYOFFICE/k8s-docs-example/actions/workflows/Release.yaml/badge.svg)](https://github.com/ONLYOFFICE/k8s-docs-example/actions/workflows/Release.yaml)

This repository contains a set of files to deploy ONLYOFFICE Docs-example into a Kubernetes cluster.

## Introduction

- To install, you need a kubernetes cluster, for installation, read the [instructions](https://kubernetes.io/docs/setup/) 
- You will need kubectl installed, you can read the instructions at this [link](https://kubernetes.io/docs/tasks/tools/).
- You will also need Helm v3. For installation, read the [instructions](https://helm.sh/docs/intro/install/)

## Installing dependencies  

### 1. Add Helm repositories

To install databases nfs server and docs-example you need to add repositories:

```bash
$ helm repo add onlyoffice https://download.onlyoffice.com/charts/stable
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

$ helm install docs-example onlyoffice/docs-example --set example.dsUrl=http://<ip>/
```

The command deploys docs-example on the Kubernetes cluster with the URL configuration of your ONLYOFFICE Docs

### 4. Uninstall ONLYOFFICE Docs-example

To uninstall/delete the `docs-example` deployment:

```bash
$ helm delete docs-example

```

### 5. Avalivable parameters

Below are the options available for configuration before deploying docs-example to a cluster: 

`persistence.storageClass` storage class name by default: `nfs`

`persistence.size` storage volume size by default: `8Gi`

`example.dsUrl` Specifies the address of your ONLYOFFICE Docs, default: `http://onlyoffice-docs-address/`

`example.replicas` docs-example replicas quantity by default: `1`

`example.containerImage` example container image name by default: `onlyoffice/docs-example:latest`

`example.imagePullPolicy` example container image pull policy by default: `IfNotPresent`

`example.resources.requests.memory` memory request by default: `128Mi`

`example.resources.requests.cpu` cpu request by default: `100m`

`example.resources.limits.memory` memory limit by default: `128Mi`

`example.resources.limits.cpu` cpu limit by defalut: `250m`

`jwt.enabled` jwt enabling parameter by default: `true`

`jwt.secret` jwt secret by default: `MYSECRET`

`jwt.header` Defines the http header that will be used to send the JSON Web Token by default: `Authorization`

`jwt.inBody` Specifies the enabling the token validation in the request body to the ONLYOFFICE Docs by default: `false` 

`jwt.existingSecret` The name of an existing secret containing variables for jwt. If not specified, a secret named `example.jwt` will be created

`service.type` docs-example service type by default: `ClusterIP`

`service.port` docs-example service port by default: `3000`

`ingress.enabled` installation of ingress service by defaule: `false`

`ingress.host` Ingress hostname for the docs-example ingress by default: `""`

`ingress.ssl.enabled` installation ssl for ingress service by default: `false`

`ingress.ssl.secret` secret name for ssl by default: `tls`

`securityContext.enabled` Enable security context for the pods by default: `false`

`securityContext.example.runAsUser` Set example containers' Security Context runAsUser by default :`1001`

`securityContext.example.runAsGroup` Set example containers' Security Context runAsGroup by default:`1001`

### 6 Expose Docs-example

By default, the docs-example is published local using default serviceType: `ClusterIP` to deploy external use the command 

```bash

$ helm install docs-example onlyoffice/docs-example --set example.dsUrl=http://<ip>/ --set service.type=LoadBalancer
```

#### 6.1 Expose Docs-example via Ingress

To install the Nginx Ingress Controller to your cluster, run the following command:

```bash
$ helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true,controller.replicaCount=2
```

See more detail about installing Nginx Ingress via Helm [here](https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx).
