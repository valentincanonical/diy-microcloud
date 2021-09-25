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

# Upload it to your worker nodes
ubuntu@node1:~$ for i in {1..3}; do lxc file push rc.local worker$i/etc/rc.local; done;

# Make it executable
ubuntu@node1:~$ for i in {1..3}; do lxc exec worker$i -- chmod +x /etc/rc.local; done;
```

### Creating the first MicroK8s node

```sh
# Log in to the first worker node, and run the commands to initiate MicroK8s
ubuntu@node1:~$ lxc shell worker1

root@worker1:~$ snap install microk8s --classic
# tip: you can already open new shells and run in // the microk8s install command on the other nodes (worker2 and worker3)

# MicroK8s is installed but not yet running
root@worker1:~$ microk8s status
microk8s is not running. Use microk8s inspect for a deeper inspection.

# We will first enable the dns and storage addons as they will be useful to most apps
root@worker1:~$ microk8s enable dns storage # this can take a few minutes


# Let's wait for MicroK8s to be up and running...
root@worker1:~$ microk8s status --wait-ready # this can take a few minutes

# done!
```

### Adding more nodes and cluster them

```sh
# Install MicroK8s on the other worker nodes of your K8s cluster
ubuntu@node1:~$ lxc exec worker2 -- snap install microk8s --classic
ubuntu@node1:~$ lxc exec worker3 -- snap install microk8s --classic

# Run the add-node command from the worker1 machine and use it to join your second node
ubuntu@node1:~$ lxc exec worker1 -- microk8s add node
ubuntu@node1:~$ lxc exec worker2 -- microk8s join <paste-token-here>

# Run the add-node command from the worker1 machine and use it to join your third node
ubuntu@node1:~$ lxc exec worker1 -- microk8s add node
ubuntu@node1:~$ lxc exec worker2 -- microk8s join <paste-token-here>

# Let's wait for the MicroK8s cluster to be up and running...
root@worker1:~$ microk8s status --wait-ready # this can take a few minutes

# done!
```

### Status, HA, Addons

```sh
root@worker1:~$ microk8s status

root@worker1:~$ microk8s addons

```