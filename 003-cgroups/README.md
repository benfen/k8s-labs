# Prerequisites
* Completed [setup](../setup/README.md)
* A single-node cluster created by `kind`

| Terminal 1 | Terminal 2 | Result |
|-|-|-|
| - | `docker exec -it kind-control-plane /bin/bash` |Initializes shell for container|
| `kubectl apply -f pod-with-resources.yml` | - | Create an httpd pod with limits |
| `kubectl get pods httpd-with-resources -o=jsonpath='{.status.containerStatuses[0].containerID}{"\n"}{.metadata.uid}{"\n"}'` | - | Fetch the ID of the pod and the container in the cluster |
| - | `ls /sys/fs/cgroup/` | View `cgroup` information |
| - | `cat /sys/fs/cgroup/memory/kubepods/pod$pod_id/$container_id/memory.limit_in_bytes` | Check the memory limit of the container |
| - | `cat /sys/fs/cgroup/cpu,cpuacct/kubepods/pod$pod_id/$container_id/cpu.cfs_quota_us` | Check the cpu limit of the container (may want to also view cpu.cfs_period_us) |
| - | `cat /sys/fs/cgroup/cpu,cpuacct/kubepods/pod$pod_id/$container_id/cgroup.procs` | View all the pids being used by the container |
| - | `pgrep -g $first_process_id` | Another way to view the processes from the last command |
| `kubectl delete -f pod-with-resources.yml` | - | Remove the httpd pod |
|-| `exit` |Exit out of the docker container|

Exercises:
* Try killing the process running the container on the node.  How does Kubernetes handle that?
* Try creating a resource-limited container that has huge requests for CPU or memory.
