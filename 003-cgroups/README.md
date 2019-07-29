# Prerequisites
* Completed [setup](../setup/README.md)

| Terminal 1 | Terminal 2 | Result |
|-|-|-|
| - | `docker exec -it kind-control-plane /bin/bash` |Initializes shell for container|
| `kubectl apply -f pod-with-resources.yml` | - | Create an httpd pod with limits |
| `kubectl get pods httpd-with-resources -o=jsonpath='{.status.containerStatuses[0].containerID}{"\n"}{.metadata.uid}{"\n"}'` | - | Fetch the ID of the pod and the container in the cluster |
| - | `ls /sys/fs/cgroup/memory/kubepods/pod$pod_id/` | View `cgroup` information for the pod |
| - | `cat /sys/fs/cgroup/memory/kubepods/pod$pod_id/$container_id/memory.limit_in_bytes` | Check the memory limit of the container |
| - | `cat /sys/fs/cgroup/cpu,cpuacct/kubepods/pod$pod_id/$container_id/cpu.cfs_period_us` | Check the cpu limit of the container |
| - | `cat /sys/fs/cgroup/cpu,cpuacct/kubepods/pod$pod_id/$container_id/cgroup.procs` | View all the pids being used by the container |
| - | `pgrep -g $first_process_id` | Another way to view the processes from the last command |
| - | `kill $first_process_id` | Kill the process running the container. |
| `kubectl get pods` | - | See that we triggered a pod restart.  If we look at the containerID backing the pod, we can see that it changed |
| `kubectl delete -f pod-with-resources.yml` | - | Remove the httpd pod |
|-| `exit` |Exit out of the docker container|
