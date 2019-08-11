# Prerequisites
* Completed [setup](../setup/README.md)

If you have an existing `kind` cluster, get rid of it with `kind delete cluster`.

```bash
# Start off by creating the regular cluster, making the default name explicit
kind create cluster --name kind
export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
# Examine the nodes
kubectl get nodes
# Examine the pods
kubectl get pods --namespace kube-system

# Now create a customized version of the cluster
kind create cluster --config kind-config.yml --name test-cluster
export KUBECONFIG="$(kind get kubeconfig-path --name="test-cluster")"
# Examine the nodes
kubectl get nodes
# Examine the pods
kubectl get pods --namespace kube-system
```
