
[< back to Step 3: Cluster the machines with LXD, your first cloud!](./step-03-lxd-cloud.md#3-cluster-the-machines-with-lxd-your-first-cloud)

# #4 Create on-demand MicroK8s clusters

_Expected Duration: 25mn_

MicroK8s is a low-ops, minimal production Kubernetes, for devs, cloud, clusters, workstations, Edge and IoT.    
In a micro cloud architecture, Kubernetes APIs make management of edge clusters easier to integrate with existing infrastructure and centralised control planes. MicroK8s is lightweight and yet features the K8s APIs, none added or removed. MicroK8s ships with with sensible defaults that ‘just work’. And from 3 nodes, MicroK8s automatically supports an highly-available configuration.

<!-- TODO: MicroK8s gif/logo? -->

## Option A: Deploy on-demand kubernetes clusters in one Juju command

Currently, there is no MicroK8s charm on [CharmHub](https://charmhub.io/), the Store for Charmed Operators. However, there is a version contributed [by @pjdc](https://launchpad.net/~pjdc/+git/charm-microk8s) a community member on Launchpad. It's just a matter of time until we get an official one!

For this workshop, we forked pjdc's version and adapted it to work on top of LXD.
We'll start by downloading this custom MicroK8s Charmed Operator, a package with all the knowledge on how to deploy a MicroK8s cluster on top of LXD.

### Register our LXD micro cloud for Model-Driven Operations

Now that we have a LXD micro cloud, we want to be able to operate it from Juju with Charmed Operators.
For that, we will need to register it.

#### Prepare the credentials to connect to our LXD micro cloud

First, let's configure remote access to our LXD micro cloud:

```sh
$ multipass shell node1

ubuntu@node1:~$ lxc remote add microcloud 192.168.64.32
Generating a client certificate. This may take a minute...
Certificate fingerprint: abc123def
ok (y/n)? y
Admin password for microcloud:
# exit with ctrl-c
```

While it might seem the command has failed, it has actually created all the certificates we need.
We will copy them to a credentials file, that we will then format for Juju to use.

```sh
# multipass shell node1
cat ~/snap/lxd/common/config/client.crt >> credentials.yaml
cat ~/snap/lxd/common/config/client.key >> credentials.yaml
cat ~/snap/lxd/common/config/servercerts/microcloud.crt >> credentials.yaml
sed -e 's/^/        /' -i credentials.yaml
```

Then use your favorite editor (e.g. `ubuntu@node1:~$ nano credentials.yaml`) to add the following yaml structure:
```yaml
credentials:
  lxdcloud:
    admin:
      auth-type: certificate
      client-cert: |
        -----BEGIN CERTIFICATE-----
        -----END CERTIFICATE-----
      client-key: |
        -----BEGIN EC PRIVATE KEY-----
        -----END EC PRIVATE KEY-----
      server-cert: |
        -----BEGIN CERTIFICATE-----
        -----END CERTIFICATE-----
```

We then need to trust the client certificate from the LXD cluster. We will do that using a juju action.

```sh
# transfer or copy and paste the client.crt file to your juju controller machine
$ multipass transfer node1:/home/ubuntu/snap/lxd/common/config/client.crt ./
$ multipass transfer ./client.crt juju:/home/ubuntu/client.crt
$ rm client.crt
# use a juju action to trust the client certificate on the LXD micro cloud cluster
$ multipass exec juju -- bash -c 'juju run-action lxd/0 add-trusted-client cert="$(cat /home/ubuntu/client.crt)" --wait'
# wait until it says the certificate has been trusted
```
<details>
    <summary>
Click here to expand the instruction for AWS cloud machines.
    </summary>

```sh
  ssh node1.aws cat /home/ubuntu/snap/lxd/common/config/client.crt | ssh juju.aws -T "cat > /home/ubuntu/client.crt"
```

</details>
</br>

Adding the remote LXD cluster should now work:
```sh
$ multipass shell node1
ubuntu@node1:~$ lxc remote add microcloud 192.168.64.32
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
ubuntu@node1:~$ juju add-cloud lxdcloud
```

#### Register as a Juju cloud
```sh
# Let's add our LXD cloud to Juju, naming it "lxdcloud"
ubuntu@node1:~$ juju add-cloud lxdcloud
# select cloud type 'lxd'
Select cloud type: lxd
# enter the API endpoint retrieved previously with the 'lxc cluster ls' command
Enter the API endpoint url for the remote LXD server: https://<node1-ip-address>:8443
... # leave other choices empty
Cloud "lxdcloud" successfully added to your local client.

# We then need to add the authentication information to administrate our LXD cloud
# We pass the credentials.yaml file previously crafted with the client certificate
ubuntu@node1:~$ juju add-credential lxdcloud -f credentials.yaml
Credential "admin" added locally for cloud "lxdcloud".

# Now we can bootstrap our LXD cloud, asking Juju to setup an agent on it
ubuntu@node1:~$ juju bootstrap lxdcloud
Creating Juju controller "lxdcloud-default" on lxdcloud/default
...
Bootstrap complete, controller "lxdcloud-default" is now available
Controller machines are in the "controller" model
Initial model "default" added

# Once the bootstrap is complete, the "status" command shows our microcloud registered
ubuntu@node1:~$ juju status
Model    Controller          Cloud/Region        Version  SLA          Timestamp
default  lxdcloud-default  lxdcloud/default  2.9.12   unsupported  13:05:13+02:00

Model "admin/default" is empty.
```

### Deploy MicroK8s clusters in one Juju command

#### Download the MicroK8s charm
```sh
# For ARM64 users
TODO: file for ARM64 users

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
kubernetes-is-easy  lxdcloud-default    lxdcloud/default    2.9.14   unsupported  18:46:16+02:00

Model "admin/kubernetes-is-easy" is empty.

# We deploy the app 'microk8s' from the local charm with force option as we use an unvalidated LXD profile
ubuntu@node1:~$ juju deploy ./microk8s_ubuntu-20.04.charm microk8s -n3 --force

# We can watch operations as they happen with 'juju status'
ubuntu@node1:~$ watch --color juju status --color
Model               Controller          Cloud/Region        Version  SLA          Timestamp
kubernetes-is-easy  lxdcloud-default   lxdcloud/default     2.9.14   unsupported  18:58:25+02:00

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

Once everything is green and active/ready, we can make use of our MicroK8s cluster!

```sh
ubuntu@node1:~$ juju ssh microk8s/0
juju@microk8sworker1:~$ microk8s status
microk8s is running
high-availability: yes
...
```

## Option B: Manually deploy a MicroK8s cluster

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
