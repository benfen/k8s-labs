# Prerequisites
* Completed [setup](../setup/README.md)

| Outside of Container | Inside of Container | Result |
|-|-|-|
| `docker exec -it kind-control-plane /bin/bash` |-|Initializes shell for container|
|-| `iptables -L` |List all the `iptables` chains|
|-| `iptables -L -t nat` |List all the `iptables` rules for the `nat` table|
| `kubectl apply -f service.yml` |-|Kubernetes will create a service for the webservers|
|-| `iptables -L -t nat \| grep example-service` |No rules are setup to route to pods since there are no pods|
|-| `iptables -L \| grep example-service` |The service rejects all requests not handled by the `nat` table (i.e. everything)|
| `kubectl apply -f controller.yml` |-|Kubernetes creates a controller with a single pod for the webserver|
|-| `iptables -L -t nat \| grep example-service` |Two rules: one to mark all incoming traffic and one to direct to the pods.|
|-| `SVC_NAME=...` |Copy name of service from iptables - format is KUBE-SVC-...|
|-| `iptables -L -t nat \| grep "Chain $SVC_NAME" -A 2` |List of all rules routing traffic to pods|
| `kubectl scale rc httpd --replicas=3` |-|An additional two webserver pods are created|
|-| `iptables -L -t nat \| grep "Chain $SVC_NAME" -A 4` |Two additional rules are created for the new pods.  Requests are divided up proportionately|
|-| `exit` |Exit out of the docker container|

```bash
# Clean up the cluster after the experiment
kubectl delete -f controller.yml
kubectl delete -f service.yml
```
