# Prerequisites
* Docker
* Go (preferably newest or 1.11+)

```bash
# Fetch the Kind package from Go and create a local cluster
GO111MODULE="on" go get sigs.k8s.io/kind@v0.4.0 && kind create cluster

# Do a quick check on the cluster for information
kubectl cluster-info
kubectl get nodes
kubectl get pods --all-namespaces
```