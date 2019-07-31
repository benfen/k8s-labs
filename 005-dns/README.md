# Prerequisites
* Completed [setup](../setup/README.md)
* A single-node cluster created by `kind`

Check to see how DNS caching could produce bad results
```bash
# Create the Kubernetes test objects
kubectl apply -f service.yml
kubectl apply -f app.yml

# Test resolving the DNS of the service
kubectl exec shell-pod getent hosts example-service
# Make a simple request to the webserver to get some simple HTML back
kubectl exec shell-pod -- wget -qO - example-service:80

# These next commands need to be run quickly so that the DNS cache does not get reset
# Make sure that the example-service DNS is in the cache
kubectl exec shell-pod -- wget -qO - example-service:80
# Destroy the service and recreate it to generate a new IP
kubectl delete -f service.yml && kubectl apply -f service.yml
# Make a request from the service; this should hang becase the DNS hasn't updated
kubectl exec shell-pod -- wget -qO - example-service:80
# Check out the new IP of the service
kubectl exec shell-pod getent hosts example-service
```

Simulate DNS pods being evicted from the cluster
```bash
# Scale down the CoreDNS deployment to simulate pods being evicted
kubectl scale deployment coredns -n kube-system --replicas=0
# Check the pod list to wait until the pods are gone
kubectl get pods # (repeat as needed)
# Attempt to make a request using DNS; this should fail because the DNS pods are gone
kubectl exec shell-pod -- wget -qO - example-service:80
# Attempt to resolve the DNS name; this should fail because the DNS pods are gone
kubectl exec shell-pod getent hosts example-service
# Restore the DNS pods
kubectl scale deployment coredns -n kube-system --replicas=2
```

Replace the existing DNS with a different provider
```bash
# Preserve this so that everything doesn't break
dns_ip=$(kubectl get service kube-dns -n kube-system -o=jsonpath='{.spec.clusterIP}')
# Remove the existing DNS service and deployment
kubectl delete -n kube-system service kube-dns
kubectl delete -n kube-system deployment coredns
# Verify that DNS is no longer functioning in the cluster (this should fail)
kubectl exec shell-pod getent hosts example-service
# Fetch the new DNS implementation and load it into the cluster
wget -O - https://raw.githubusercontent.com/kubernetes/kubernetes/v1.15.1/cluster/addons/dns/kube-dns/kube-dns.yaml.sed \
    | sed -e "s/\$DNS_SERVER_IP/$dns_ip/g" \
    | sed -e "s/\$DNS_MEMORY_LIMIT/100Mi/g" \
    | sed -e "s/\$DNS_DOMAIN/cluster.local/g" \
    | kubectl apply -f -
# Wait for the pods to start up
kubectl get pods -n kube-system
# Verify that DNS is once again functioning in the cluster
kubectl exec shell-pod getent hosts example-service
```
