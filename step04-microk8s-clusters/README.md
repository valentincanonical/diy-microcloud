# On-demand Micro Kubernetes Clusters

## Manual Installation

Read more on [the microk8s.io website](https://microk8s.io/docs/lxd).

### Pre-req
#### Configuring the MicroK8s LXD profile

```sh
# Create a new LXD profile for MicroK8s
ubuntu@node1:~$ lxc profile create microk8s
Profile microk8s created

# Download the ZFS profile template from https://microk8s.io/docs/lxd
ubuntu@node1:~$ wget https://raw.githubusercontent.com/ubuntu/microk8s/master/tests/lxc/microk8s-zfs.profile -O microk8s.profile
microk8s.profile                  100%[============================================================>]     849  --.-KB/s    in 0s      

# Apply it to the previously created LXD profile
ubuntu@node1:~$ cat microk8s.profile | lxc profile edit microk8s

# Clean behind us - done!
ubuntu@node1:~$ rm microk8s.profile
```

#### Launching the worker/master nodes machines

<!-- ToDo: default to command using Juju (in a new model) to create machines -->
```sh
# Let's iterate to create three LXD containers as three worker/master nodes
ubuntu@node1:~$ for i in {1..3}; do lxc launch -p default -p microk8s ubuntu:20.04 worker$i; done;
Starting worker1
Starting worker2
Starting worker3

ubuntu@node1:~$ lxc ls      
+---------+---------+---------------------+-------+-----------+-----------+
|  NAME   |  STATE  |        IPV4         |  IPV6 |   TYPE    | SNAPSHOTS |
+---------+---------+---------------------+-------+-----------+-----------+
| worker1 | RUNNING | 10.204.8.3 (eth0)   |       | CONTAINER | 0         |
+---------+---------+---------------------+-------+-----------+-----------+
| worker2 | RUNNING | 10.204.8.56 (eth0)  |       | CONTAINER | 0         |
+---------+---------+---------------------+-------+-----------+-----------+
| worker3 | RUNNING | 10.204.8.170 (eth0) |       | CONTAINER | 0         |
+---------+---------+---------------------+-------+-----------+-----------+

# One more LXD specific configuration for the AppArmor profiles, as per the docs on https://microk8s.io/docs/lxd
# Create 'rc.local' file on your node machine with the following content
ubuntu@node1:~$ cat > rc.local <<EOF
#!/bin/bash


apparmor_parser --replace /var/lib/snapd/apparmor/profiles/snap.microk8s.*
exit 0
EOF

# Upload it to your worker nodes and make it executable
ubuntu@node1:~$ for i in {1..3}; do lxc file push rc.local worker$i/etc/rc.local; lxc exec worker$i -- chmod +x /etc/rc.local; done;
```

### Creating the first MicroK8s node

```sh
# Log in to the first worker node, and run the commands to initiate MicroK8s
ubuntu@node1:~$ lxc exec worker1 -- snap install microk8s --classic
microk8s (1.21/stable) v1.21.5 from Canonical✓ installed

# MicroK8s is installed but not yet running
ubuntu@node1:~$ lxc exec worker1 -- microk8s status
microk8s is not running. Use microk8s inspect for a deeper inspection.

# FOR THE MULTIPASS OPTION ONLY - disable some kernal calls as a workaround to
# the too many virtualisation layers --> vm --> lxd --> snap --> kubernetes
ubuntu@node1:~$ lxc exec worker1 -- bash -c 'echo "--conntrack-max-per-core=0" >> /var/snap/microk8s/current/args/kube-proxy && 
systemctl restart snap.microk8s.daemon-kubelite'

# We will need dns and storage modules for clustering and further deployments
ubuntu@node1:~$ lxc exec worker1 -- microk8s enable dns storage
DNS is enabled
Storage will be available soon

# MicroK8s might take up to 5mn to get up and ready...
ubuntu@node1:~$ lxc exec worker1 -- microk8s status --wait-ready
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
```

### Adding more worker nodes

Repeat the following commands for each MicroK8s worker node you cant to configure:
```sh
i=2
lxc exec worker$i -- snap install microk8s --classic
lxc exec worker$i -- bash -c 'echo "--conntrack-max-per-core=0" >> /var/snap/microk8s/current/args/kube-proxy && systemctl restart snap.microk8s.daemon-kubelite'
```

### Cluster your MicroK8s nodes

```sh
# Run the add-node command from the worker1 machine and use it to join your second node
ubuntu@node1:~$ lxc exec worker1 -- microk8s add-node
ubuntu@node1:~$ lxc exec worker2 -- microk8s join <paste-token-here>

# Run the add-node command from the worker1 machine and use it to join your third node
ubuntu@node1:~$ lxc exec worker1 -- microk8s add-node
ubuntu@node1:~$ lxc exec worker3 -- microk8s join <paste-token-here>

# Let's wait for the MicroK8s cluster to be up and running...
ubuntu@node1:~$ lxc exec worker1 -- microk8s status --wait-ready # this can take a few minutes

# done!
```

You can run some status commands to show around and make sure everything is up and ready:
```sh
ubuntu@node1:~$ lxc shell worker1

root@worker1:~$ microk8s status
microk8s is running
high-availability: yes
  datastore master nodes: 240.64.34.187:19001 240.64.33.180:19001 240.64.32.162:19001
  datastore standby nodes: none
addons:
  enabled:
    dns                  # CoreDNS
    ha-cluster           # Configure high availability on the current node
    storage              # Storage class; allocates storage from host directory
  disabled:
    ambassador           # Ambassador API Gateway and Ingress
    cilium               # SDN, fast with full network policy
    dashboard            # The Kubernetes dashboard
    fluentd              # Elasticsearch-Fluentd-Kibana logging and monitoring
    gpu                  # Automatic enablement of Nvidia CUDA
    helm                 # Helm 2 - the package manager for Kubernetes
    helm3                # Helm 3 - Kubernetes package manager
    host-access          # Allow Pods connecting to Host services smoothly
    ingress              # Ingress controller for external access
    istio                # Core Istio service mesh services
    jaeger               # Kubernetes Jaeger operator with its simple config
    keda                 # Kubernetes-based Event Driven Autoscaling
    knative              # The Knative framework on Kubernetes.
    kubeflow             # Kubeflow for easy ML deployments
    linkerd              # Linkerd is a service mesh for Kubernetes and other frameworks
    metallb              # Loadbalancer for your Kubernetes cluster
    metrics-server       # K8s Metrics Server for API access to service metrics
    multus               # Multus CNI enables attaching multiple network interfaces to pods
    openebs              # OpenEBS is the open-source storage solution for Kubernetes
    openfaas             # openfaas serverless framework
    portainer            # Portainer UI for your Kubernetes cluster
    prometheus           # Prometheus operator for monitoring and logging
    rbac                 # Role-Based Access Control for authorisation
    registry             # Private image registry exposed on localhost:32000
    traefik              # traefik Ingress controller for external access

root@worker1:~$ microk8s kubectl get all -A
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   pod/hostpath-provisioner-5c65fbdb4f-27mmn     1/1     Running   0          14m
kube-system   pod/calico-kube-controllers-f7868dd95-9nslg   1/1     Running   0          53m
kube-system   pod/coredns-7f9c69c78c-9ggbt                  1/1     Running   0          14m
kube-system   pod/calico-node-49b6q                         1/1     Running   3          4m12s
kube-system   pod/calico-node-qllhh                         1/1     Running   0          2m58s
kube-system   pod/calico-node-qsnvm                         1/1     Running   0          2m39s

NAMESPACE     NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.152.183.1    <none>        443/TCP                  53m
kube-system   service/kube-dns     ClusterIP   10.152.183.10   <none>        53/UDP,53/TCP,9153/TCP   51m

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/calico-node   3         3         3       3            3           kubernetes.io/os=linux   53m

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns                   1/1     1            1           51m
kube-system   deployment.apps/hostpath-provisioner      1/1     1            1           51m
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           53m

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-7f9c69c78c                  1         1         1       14m
kube-system   replicaset.apps/hostpath-provisioner-5c65fbdb4f     1         1         1       14m
kube-system   replicaset.apps/calico-kube-controllers-f7868dd95   1         1         1       53m
```
