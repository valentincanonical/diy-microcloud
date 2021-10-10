
[< back to Step 3: Cluster the machines with LXD, your first cloud!](./step-03-lxd-cloud.md#3-cluster-the-machines-with-lxd-your-first-cloud)

# #4 Create on-demand MicroK8s clusters

_Expected Duration: 25mn_

MicroK8s is a low-ops, minimal production Kubernetes, for devs, cloud, clusters, workstations, Edge and IoT.    
In a micro cloud architecture, Kubernetes APIs make management of edge clusters easier to integrate with existing infrastructure and centralised control planes. MicroK8s is lightweight and yet features the K8s APIs, none added or removed. MicroK8s ships with with sensible defaults that ‘just work’. And from 3 nodes, MicroK8s automatically supports an highly-available configuration.

<!-- 

TODO: Prebuild charms for multi-arch and provide them in ./precompiled to enable the Juju option
TODO: Fix the MicroK8s community charm

#### Register your LXD cloud with Juju

<img alt="LXD cloud" src="./img/checkpoint-03.png" width="600" />

Now that we have a LXD micro cloud, we want to be able to operate it from Juju with Charmed Operators. For that, we will need to register it:

[Tutorial Step 3, part2: Register your LXD micro cloud with Juju](./step03-juju-bootstrap/README.md#model-driven-operated-micro-cloud-with-juju).

```
ubuntu@juju:~$ juju status
Model    Controller          Cloud/Region        Version  SLA          Timestamp
default  microcloud-default  microcloud/default  2.9.12   unsupported  13:05:13+02:00

Model "admin/default" is empty.
```

#### Option A: Deploy on-demand kubernetes clusters with Juju

Currently, there is no MicroK8s charm on [CharmHub](https://charmhub.io/), the Store for Charmed Operators. However, there is a version contributed [by @pjdc](https://launchpad.net/~pjdc/+git/charm-microk8s) a community member on Launchpad. It's just a matter of time until we get an official one!

For this workshop, we forked pjdc's version and adapted it to work on top of LXD. We'll start by cloning my repository with the MicroK8s' Charmed Operator, a package with all the knowledge on how to deploy a MicroK8s cluster.


TODO: precompile for more platforms and compile instructions collapsed
```sh
# Clone the modified Charmed Operator where you installed Juju at Step #3
ubuntu@node1:~$ git clone -b fix/lxd https://git.launchpad.net/~valentinviennot/+git/charm-microk8s
or 
ubuntu@node1:~$ wget https://raw.githubusercontent.com/valentincanonical/diy-microcloud/main/precompiled/microk8s_ubuntu-20.04-amd64.charm -O microk8s_ubuntu-20.04.charm
```

In order, we will:
- create a new model, a clean space to isolate our work;
- deploy four microk8s nodes with a simple juju command;
- sip a cocktail 🍹 while Juju does all the work for us (launch four LXD containers, install microk8s on each of them, and cluster them together).

```sh
ubuntu@node1:~$ juju models
Controller: microcloud-default
Model                Cloud/Region        Type  Status     Machines  Access  Last connection
controller           microcloud/default  lxd   available         1  admin   just now
default              microcloud/default  lxd   available         0  admin   8 minutes ago
kubernetes-is-easy*  microcloud/default  lxd   available         0  admin   28 seconds ago

ubuntu@node1:~$ juju add-model kubernetes-is-easy

ubuntu@node1:~$ juju status
Model               Controller          Cloud/Region        Version  SLA          Timestamp
kubernetes-is-easy  microcloud-default  microcloud/default  2.9.14   unsupported  18:46:16+02:00

Model "admin/kubernetes-is-easy" is empty.

# We deploy the app 'microk8s' from the local charm with force option as we use an unvalidated LXD profile
ubuntu@node1:~$ juju deploy ./microk8s_ubuntu-20.04.charm microk8s -n3 --force

# We can watch operations as they happen with 'juju status'
ubuntu@node1:~$ watch --color juju status --color
Model               Controller          Cloud/Region        Version  SLA          Timestamp
kubernetes-is-easy  microcloud-default  microcloud/default  2.9.14   unsupported  18:58:25+02:00

App       Version  Status   Scale  Charm     Store  Channel  Rev  OS      Message
microk8s           waiting    0/3  microk8s  local             0  ubuntu  waiting for machine

Unit        Workload  Agent       Machine  Public address  Ports  Message
microk8s/0  waiting   allocating  0        240.33.0.11            waiting for machine
microk8s/1  waiting   allocating  1                               waiting for machine
microk8s/2  waiting   allocating  2                               waiting for machine

Machine  State    DNS          Inst id        Series  AZ  Message
0        pending  240.33.0.11  juju-ab5f48-0  focal       Running
1        pending               pending        focal       Creating container
2        pending               pending        focal
```

#### Option B: Manually deploy a MicroK8s cluster

If you want to see what is happening under the hood, you can manually start LXD containers and set up MicroK8s. We would recommend using [the Juju option](#deploy-on-demand-kubernetes-clusters-with-juju) to save some time and uncover the power of Charmed Operators, but you're also good to go with this option. Installing MicroK8s is only the matter of one "snap install microk8s" command and a "microk8s add/join" per machine to cluster your nodes together.

[Click here for the instructions to manually create MicroK8s clusters.](./step04-microk8s-clusters/README.md#manual-installation)

-->

Read more on [the microk8s.io website](https://microk8s.io/docs/lxd).

## Prepare the LXD worker nodes

We are going to install MicroK8s on top of our micro cloud, and therefore will need a few LXD machines to use as worker nodes.

### Configure the MicroK8s LXD profile

```sh
#!/bin/bash
$ multipass shell node1

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

### Launch the worker nodes

```sh
#!/bin/bash

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
```

One more LXD specific configuration for the AppArmor profile, as per the docs on https://microk8s.io/docs/lxd:

```sh
#!/bin/bash

# Create 'rc.local' file on your node machine with the following content
ubuntu@node1:~$ cat > rc.local <<EOF
#!/bin/bash


apparmor_parser --replace /var/lib/snapd/apparmor/profiles/snap.microk8s.*
exit 0
EOF

# Upload it to your worker nodes and make it executable
ubuntu@node1:~$ for i in {1..3}; do lxc file push rc.local worker$i/etc/rc.local; lxc exec worker$i -- chmod +x /etc/rc.local; done;
```

## Create the first MicroK8s instance

```sh
# Log in to the first worker node, and run the command to install MicroK8s
ubuntu@node1:~$ lxc exec worker1 -- snap install microk8s --classic
microk8s (1.21/stable) v1.21.5 from Canonical✓ installed

# MicroK8s is installed but not yet running
ubuntu@node1:~$ lxc exec worker1 -- microk8s status
microk8s is not running. Use microk8s inspect for a deeper inspection.

# FOR THE MULTIPASS OPTION ONLY ---
# disable some kernal calls as a workaround to the too many virtualisation layers (--> vm --> lxd --> snap --> kubernetes)
ubuntu@node1:~$ lxc exec worker1 -- bash -c 'echo "--conntrack-max-per-core=0" >> /var/snap/microk8s/current/args/kube-proxy && 
systemctl restart snap.microk8s.daemon-kubelite'
# ---

# We will need the dns and storage modules for clustering, and further application deployments
ubuntu@node1:~$ lxc exec worker1 -- microk8s enable dns storage
DNS is enabled
Storage will be available soon

# MicroK8s might take up to 5mn to get up and ready... done!
ubuntu@node1:~$ lxc exec worker1 -- microk8s status --wait-ready
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
```

## Add more MicroK8s worker nodes

Repeat the following commands for each MicroK8s worker node you want to configure.
In these instructions, we are using 3 worker nodes to demonstrate the high-availability configuration of MicroK8s.

> If you feel that your machine is already under pressure... or that you're a bit behind on timing:     
> you can stick to a single-node MicroK8s (and jump [to the next step](./step-05-micro-cloud-native.md#5-run-cloud-native-applications-at-the-edge-with-micro-clouds)).

```sh
i=2
lxc exec worker$i -- snap install microk8s --classic
lxc exec worker$i -- bash -c 'echo "--conntrack-max-per-core=0" >> /var/snap/microk8s/current/args/kube-proxy && systemctl restart snap.microk8s.daemon-kubelite'
# repeat for i=3...
```

## Cluster your MicroK8s nodes

Now that you have deployed multiple MicroK8s instances, the next logical step is to cluster them together.
Thanks to the micro cloud setup, it's trivial to organise them as you wish and have different kubernetes clusters running in parallel.

```sh
# Run the add-node command from the worker1 machine and use it to join your second node
ubuntu@node1:~$ lxc exec worker1 -- microk8s add-node
  <copy-join-token>

ubuntu@node1:~$ lxc exec worker2 -- microk8s join <paste-join-token-here>

# Run the add-node command from the worker1 machine and use it to join your third node
ubuntu@node1:~$ lxc exec worker1 -- microk8s add-node
  <copy-join-token>

ubuntu@node1:~$ lxc exec worker3 -- microk8s join <paste-token-here>

# Let's wait for the MicroK8s cluster to be up and running...
ubuntu@node1:~$ lxc exec worker1 -- microk8s status --wait-ready

# This can take a few minutes to complete... done!
# Note: some users experienced timeouts at this stage, don't panic and simply try it again...
# your machine might be under pressure running three kubernetes instances in parallel!
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

---

> **Checkpoint #4: MicroK8s cluster on LXD, up and running.**

```sh
root@worker1:~$ microk8s status
microk8s is running
high-availability: yes
  datastore master nodes: 240.64.34.187:19001 240.64.33.180:19001 240.64.32.162:19001
  datastore standby nodes: none
```

<img alt="MicroK8s cluster on LXD" src="./img/checkpoint-04.png" width="600" />

---

[Next step (5/5): Run cloud-native applications at the edge with micro clouds >](./step-05-micro-cloud-native.md#5-run-cloud-native-applications-at-the-edge-with-micro-clouds)
