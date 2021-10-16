
[< back to Step 3: Cluster the machines with LXD, your first cloud!](./step-03-lxd-cloud.md#3-cluster-the-machines-with-lxd-your-first-cloud)

# #4 Create on-demand MicroK8s clusters

_Expected Duration: 25mn_

MicroK8s is a low-ops, minimal production Kubernetes, for devs, cloud, clusters, workstations, Edge and IoT.    
In a micro cloud architecture, Kubernetes APIs make management of edge clusters easier to integrate with existing infrastructure and centralised control planes. MicroK8s is lightweight and yet features the K8s APIs, none added or removed. MicroK8s ships with with sensible defaults that ‘just work’. And from 3 nodes, MicroK8s automatically supports an highly-available configuration.

<img alt="MicroK8s" src="https://assets.ubuntu.com/v1/e63c5d75-microk8s.svg" width="100" />

## Option A: Deploy on-demand kubernetes clusters in one Juju command

Currently, there is no MicroK8s charm on [CharmHub](https://charmhub.io/), the Store for Charmed Operators. However, there is a version contributed [by @pjdc](https://launchpad.net/~pjdc/+git/charm-microk8s) a community member on Launchpad. It's just a matter of time until we get an official one!

For this workshop, we forked pjdc's version and adapted it to work on top of LXD.
We'll start by downloading this custom MicroK8s Charmed Operator, a package with all the knowledge on how to deploy a MicroK8s cluster on top of LXD.

### Register our LXD micro cloud for Model-Driven Operations

Now that we have a LXD micro cloud, we want to be able to operate it from Juju with Charmed Operators.
For that, we need to register it.

#### Prepare the credentials to connect to our LXD micro cloud

First, let's configure remote access to our LXD micro cloud.

If you deployed your LXD cluster with Juju, it doesn't have an administrator password and needs to be accessed with certificate-based authentication.

We will run the `lxc remote add` command in order to generate a new client certificate.

```sh
$ multipass shell node1

ubuntu@node1:~$ lxc remote add microcloud <ip-node1>
Generating a client certificate. This may take a minute...
Certificate fingerprint: abc123def
ok (y/n)? y
Admin password for microcloud:# exit with ctrl-c

ubuntu@node1:~$ logout
```

While it might seem the command has failed, it has actually created all we need.

Let's copy our newly generated client certificate to the `juju` controller machine:

```sh
# transfer or copy and paste the client.crt file to your juju controller machine
$ multipass transfer node1:/home/ubuntu/snap/lxd/common/config/client.crt ./
$ multipass transfer ./client.crt juju:/home/ubuntu/client.crt
$ rm client.crt
```

<details>
    <summary>
Click here to expand the instruction for AWS cloud machines.
    </summary>

From your host:

```sh
  ssh node1.aws cat /home/ubuntu/snap/lxd/common/config/client.crt | ssh juju.aws -T "cat > /home/ubuntu/client.crt"
```

</details>
</br>

We need to instruct our LXD cluster to trust our client certificate so that Juju can use it to operate our micro cloud.

We will do that using a Juju action. Juju actions are day-2 operations built into a Charmed Operator.
<!-- or, from node1: lxc config trust add /home/ubuntu/snap/lxd/common/config/client.crt -->

```sh
# use a juju action to trust the client certificate on the LXD micro cloud cluster
$ multipass shell juju

ubuntu@juju:~$ juju run-action lxd/0 add-trusted-client cert="$(cat /home/ubuntu/client.crt)" --wait
# wait until it says the certificate has been trusted

$ logout
```

Adding the remote LXD cluster should now work:

```sh
$ multipass shell node1
ubuntu@node1:~$ lxc remote add microcloud <ip-node1>
ubuntu@node1:~$ lxc remote switch microcloud
ubuntu@node1:~$ lxc cluster ls
+-------+----------------------------+----------+--------+-------------------+--------------+
| NAME  |            URL             | DATABASE | STATE  |      MESSAGE      | ARCHITECTURE |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node1 | https://192.168.64.32:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node2 | https://192.168.64.34:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node3 | https://192.168.64.33:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+
```

We can now securely access our LXD cluster remotely.

#### Install Juju on the micro cloud

```sh
ubuntu@node1:~$ sudo snap install juju --classic
```

#### Register as a Juju cloud

We don't need to explictly `juju add-cloud` nor `juju add-credential microcloud` as we already did the remote configuration earlier with the certificate and the `lxc remote add` command. [Read more about Juju clouds](https://juju.is/docs/olm/clouds).

```sh
# Let's bootstrap our LXD cloud, asking Juju to setup an agent on it
ubuntu@node1:~$ juju bootstrap microcloud
Creating Juju controller "microcloud-default" on microcloud/default
...
Bootstrap complete, controller "microcloud-default" is now available
Controller machines are in the "controller" model
Initial model "default" added

# Once the bootstrap is complete, the "status" command shows our microcloud registered
ubuntu@node1:~$ juju status
Model    Controller          Cloud/Region        Version  SLA          Timestamp
default  microcloud-default  microcloud/default  2.9.12   unsupported  13:05:13+02:00

Model "admin/default" is empty.
```

### Deploy MicroK8s clusters in one Juju command

#### Download the MicroK8s charm
```sh
# For ARM64 users
ubuntu@node1:~$ wget https://raw.githubusercontent.com/valentincanonical/diy-microcloud/main/precompiled/microk8s_ubuntu-20.04-arm64.charm -O microk8s_ubuntu-20.04.charm

# For AMD64 users
ubuntu@node1:~$ wget https://raw.githubusercontent.com/valentincanonical/diy-microcloud/main/precompiled/microk8s_ubuntu-20.04-amd64.charm -O microk8s_ubuntu-20.04.charm

# For Rapsberry Pi ARM64 users
TODO file for Rpi users
```

<details>
    <summary>
or, you can also compile it yourself for your platform (click to expand the instructions).
    </summary>

```sh
    # on a new, clean machine
    git clone -b fix/lxd https://git.launchpad.net/~valentinviennot/+git/charm-microk8s
    sudo snap install charmcraft --classic
    sudo lxd init --auto
    cd ./charm-microk8s
    charmcraft build
```

</details>
</br>

#### Deploy with Juju

In order, we will:
- create a new model, a clean space to isolate our work;
- deploy four microk8s nodes with a simple juju command;
- sip a cocktail 🍹 while Juju does all the work for us (launch four LXD containers, install microk8s on each of them, and cluster them together).

```sh
ubuntu@node1:~$ juju add-model kubernetes-is-easy

ubuntu@node1:~$ juju status
Model               Controller          Cloud/Region        Version  SLA          Timestamp
kubernetes-is-easy  microcloud-default  microcloud/default  2.9.14   unsupported  18:46:16+02:00

Model "admin/kubernetes-is-easy" is empty.
```

We deploy the app `microk8s` from the locally downloaded MicroK8s charm.

> The `--force` option is required as we use an unvalidated LXD profile.    
> For ARM64 users, the `--constraints arch=arm64` option tells juju to provision ARM64 virtual machines.

```sh
ubuntu@node1:~$ juju deploy ./microk8s_ubuntu-20.04.charm microk8s -n3 --force
# For ARM users:
# juju deploy ./microk8s_ubuntu-20.04.charm microk8s -n3 --force --constraints arch=arm64

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

<!--
TODO: tell the story
- what is happening
- what have we achieved
- what are we going to do next?
-->
Once everything is green and active/ready, we can make use of our MicroK8s cluster!

```sh
ubuntu@node1:~$ juju ssh microk8s/0
juju@microk8sworker1:~$ microk8s status
microk8s is running
high-availability: yes
...
```

## Option B: Manually deploy MicroK8s clusters

If you want to see what is happening under the hood, you can manually start LXD containers and set up MicroK8s. We would recommend using [the Juju option](#deploy-on-demand-kubernetes-clusters-with-juju) to save some time and uncover the power of Charmed Operators, but you're also good to go with this option. Installing MicroK8s is only the matter of one "snap install microk8s" command and a "microk8s add/join" per machine to cluster your nodes together.

[Click here for the instructions to manually create MicroK8s clusters.](./step04-microk8s-clusters/README.md#manual-installation)

Read more on [the microk8s.io website](https://microk8s.io/docs/lxd).

---

> **Checkpoint #4: MicroK8s cluster on LXD, up and running.**

```sh
root@worker1:~$ microk8s status
microk8s is running
high-availability: yes
  datastore master nodes: 240.64.34.187:19001 240.64.33.180:19001 240.64.32.162:19001
  datastore standby nodes: none
...
```

<img alt="MicroK8s cluster on LXD" src="./img/checkpoint-04.png" width="600" />

---

[Next step (5/5): Run cloud-native applications at the edge with micro clouds >](./step-05-micro-cloud-native.md#5-run-cloud-native-applications-at-the-edge-with-micro-clouds)
