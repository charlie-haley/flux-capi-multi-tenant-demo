# Flux, Cluster API, Multi-Tenancy Demo

## Requirements
- [clusterctl](https://github.com/kubernetes-sigs/cluster-api/tree/main/cmd/clusterctl)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

## 📂 Folder Structure
```bash
├── clusters            # These manifests are used in provisioning clusters using Cluster API.
│   ├── cluster0.yaml
│   ├── cluster1.yaml
│   └── cluster2.yaml
└── manifests           # These manifests are deployed onto the various clusters, including workload and management.
    ├── _base
    │   ├── base-namespace
    │   └── ...
    ├── cluster0
    │   ├── foo-namespace
    │   └── ...
    ├── cluster1
    │   ├── monitoring
    │   └── ...
    ├── cluster2
    │   └── ...
    └── management
    │   ├── flux-system # The manifests needed to deploy and configure Flux
    │   └── ...
```

## 🚧 Provisioning management cluster
This is one of the initial manual tasks needed, this should only need doing once! Once the management cluster is running it will handle all deployments and provisioning of new clusters.

```bash
# create cluster using k3d - you can also minikube, kind etc.
k3d cluster create management-test

# clusterctl init (! this uses your current kube context !)
clusterctl init --infrastructure docker
```

Once the management cluster is running, you can bootstrap Flux onto the cluster to deploy all the relevant manifests.
```bash
flux check --pre
kubectl create namespace flux-system

# bootstrap flux onto the management cluster - you may need to run this twice due to race conditions
kubectl apply -k ./manifests/management
```

## ✅ Validating

```bash
kubectl get clusters -A

NAMESPACE   NAME       AGE     PHASE
default     cluster0   5m58s   Provisioned
default     cluster1   5m58s   Provisioned
default     cluster2   5m58s   Provisioned
```
