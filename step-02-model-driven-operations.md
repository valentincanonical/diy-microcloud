[< back to Step 1: Prepare the bare metal nodes](./step-01-prepare-bare-nodes.md#1-prepare-the-bare-metal-nodes)

# #2 Register for model-driven operations

_Expected duration: 10mn_

## What are Model-Driven Operations?

"[Model-Driven Operations](https://juju.is/model-driven-operations-manifesto), say what?" The concept is relatively easy yet extremely powerful. Imagine we were discussing over the phone, and you suddenly wanted me to draw you a sheep; you would have two options:
- Guide me, step by step, trying to be as specific as possible - "draw four small vertical lines, now draw a circle on top..."
- Or rely on our shared knowledge of concepts and simply ask, "can you draw me a sheep, please."

I promise you; the first option won't give you anything even close to a sheep... and it will take _a lot of time_. And if someone else was listening to our conversation and trying to follow the drawing instructions, I am sure theirs wouldn't look anything like mine... nor as a sheep.

<p align="center">
<img alt="Better sheep with Model Driven Operations." src="./img/sheeps.jpg" width="400" />
</p>

<!-- (Ok, not good at drawings... but you can agree the result is still better when relying on shared concepts! :) ) -->

Now imagine I am a server, and there are thousands of us with slightly different configurations (different cloud, platform, architecture...). Wouldn't it be great if you could just teach us what a database is? And teach us how it relates to other applications to provide persistent storage? That's what Model Driven Operations are about!    

With Charmed Operators, you can package concepts and operational knowledge. With Juju, you can apply this knowledge with declarative queries, politely asking for what you need. "Please Juju, deploy this and that. Also, please Juju relate this with that." And that's it; you have a web server deployed on multiple clouds with a database properly configured and related to your web application! If you're interested, there's a lot of exciting and fun reads [on Juju.is](https://juju.is/blog).

<p align="center">
<img alt="Web Infrastructure deployed with Juju and Charmed Operators." src="./img/juju-web-bundle.png" width="400" />
</p>

Check out [this great example](https://jaas.ai/web-infrastructure-in-a-box/bundle/10) of Charmed Operators and relations (relations diagram above). Also, if you're curious to understand how JuJu can help brewing coffee with Kubernetes, [here's a nice treat](https://juju.is/brew)!

<!-- ToDo: Add link to the Edge MDO blog entry on Ubuntu.com (TBD) -->

## Register the bare nodes with Juju

> You can decide to skip this step and [jump to the next section](./step-03-lxd-cloud.md#option-b-manually-configure-lxd-cluster) if you're going for the manual installation instead of using Juju and Charmed Operators.

### Launch the Juju controller machine

We need an additional controller machine to operate our physical nodes. This can be any machine that has network access to your micro cloud nodes. Let's simply launch an new tiny VM with Multipass.

```sh
multipass launch --name juju --mem 2G --disk 5G --cpus 2
```

> If you're following the tutorial using Rapsberry Pis, you can operate this step directly from your host. Please adapt the commands.    
> If you chose to use cloud machines, you should already have it ready. Otherwise, simply launch a new one with minimal configuration.

You should now be in a situation with at least three virtual or physical machines dedicated as micro cloud nodes and a small controller machine for Juju.

```sh
$ multipass list
Name                    State             IPv4             Image
juju                    Running           192.168.64.4     Ubuntu 20.04 LTS
node1                   Running           192.168.64.32    Ubuntu 20.04 LTS
node2                   Running           192.168.64.33    Ubuntu 20.04 LTS
node3                   Running           192.168.64.34    Ubuntu 20.04 LTS
```

### Install Juju

The command to install [Juju](https://juju.is/) is pretty straightforward, and shouldn't take long!

```sh
$ multipass shell juju
ubuntu@juju:~$ sudo snap install juju --classic
```

<!-- TODO: refer back to the VPN issues doc -->

### Configure SSH access

You'll need SSH access from the Juju controller machine to your micro cloud nodes in order to operate them with Juju.
Let's generate an SSH key pair on the "Juju" machine and add the public key as an authorized host to the micro cloud nodes.

```sh
$ multipass shell juju
# Generate a new SSH key pair (presse enter to accept all the default parameters)
ubuntu@juju:~$ ssh-keygen

# Allow the Juju controller to ssh-login into itself
ubuntu@juju:~$ echo "$(cat .ssh/id_rsa.pub)" >> .ssh/authorized_keys

ubuntu@juju:~$ logout
# Switch machines and repeat the operation for the other nodes
```
If you're using Multipass, you can run this bash line from the host to add the public key in each node's authorized keys file:
```sh
#!/bin/bash
$ SSHJUJU="$(multipass exec juju -- cat /home/ubuntu/.ssh/id_rsa.pub)" && for i in {1..3}; do multipass exec node$i -- bash -c "echo $SSHJUJU >> /home/ubuntu/.ssh/authorized_keys"; done;
```
<details>
    <summary>
Click here to expand the instruction for AWS cloud machines.
    </summary>

```sh
  SSHJUJU="$(ssh juju.aws -- cat /home/ubuntu/.ssh/id_rsa.pub)" && for i in {1..3}; do echo "$SSHJUJU" | ssh node$i.aws -T "cat >> /home/ubuntu/.ssh/authorized_keys"; done;
```

</details>
</br>

### Create bare cloud

Now that we have everything in place, let's create a Juju bare cloud to control our micro cloud nodes.

> Note the 'maas' option in the 'Cloud Types' list.
> If we used MAAS to provision our nodes, these steps would be much easier!
> We'll keep that in mind for the next time.

<!-- ToDo: explain the concept, what is happening behind the scenes, high level -->

```sh
$ multipass shell juju
# Make sure to write down the IP address of your Juju controller
ubuntu@juju:~$ ip a|grep "inet "
  inet 127.0.0.1/8 scope host lo
  inet 192.168.64.4/24 brd 192.168.64.255 scope global dynamic enp0s2

ubuntu@juju:~$ juju add-cloud bare

Cloud Types
  lxd
  maas
  manual
  openstack
  vsphere
# Select the "manual" cloud type
Select cloud type: manual

# Build the SSH connection string from the 'juju' machine IP address
Enter the ssh connection string for controller, username@<hostname or IP> or <hostname or IP>:
# in our example, this would be ubuntu@192.168.64.4
ubuntu@<IP-juju-controller-machine>

Cloud "bare" successfully added to your local client.
```

Once we have registered our bare metal cloud, we need to bootstrap it.
The bootstrap operation will launch the Juju processes on the controller machine.
```sh
ubuntu@juju:~$ juju bootstrap bare

Creating Juju controller "bare-default" on bare/default
...
Initial model "default" added
```

### Add the micro cloud nodes to your Juju bare cloud

<!-- 
TODO: add some backup materials here during the wait
-->

```sh
# Make sure to write down the IP addresses of your micro cloud nodes
$ multipass list
Name                    State             IPv4             Image
juju                    Running           192.168.64.4     Ubuntu 20.04 LTS
node1                   Running           192.168.64.32    Ubuntu 20.04 LTS
node2                   Running           192.168.64.33    Ubuntu 20.04 LTS
node3                   Running           192.168.64.34    Ubuntu 20.04 LTS

$ multipass shell juju

# First create a model to contain our machines and LXD application
ubuntu@juju:~$ juju add-model microcloud

# Add your micro cloud nodes as machines to the bare cloud
# We use the '--verbose' option for more visibility
ubuntu@juju:~$ juju add-machine --show-log -v ssh:ubuntu@<IP-node1>
created machine 0

ubuntu@juju:~$ juju add-machine --show-log -v ssh:ubuntu@<IP-node2>
created machine 1

ubuntu@juju:~$ juju add-machine --show-log -v ssh:ubuntu@<IP-node3>
created machine 2

# ...done! Juju now sees all the machines in our micro cloud, we are ready to deploy applications on top

ubuntu@juju:~$ juju status                                                          
Model    Controller    Cloud/Region  Version  SLA  
default  bare-default  bare/default  2.9.14   unsupported

Machine  State    DNS            Inst id               Series  AZ  Message
0        started  192.168.64.32  manual:192.168.64.32  focal       Manually provisioned machine
1        started  192.168.64.33  manual:192.168.64.33  focal       Manually provisioned machine
2        started  192.168.64.34  manual:192.168.64.34  focal       Manually provisioned machine
```

---

> **Checkpoint #2: Juju bare cloud with micro cloud nodes added as machines.**

<img alt="LXD cloud registered as a Juju cloud." src="./img/checkpoint-02.png" width="600" />

---

[Next step (3/5): Cluster the machines with LXD, your first cloud! >](./step-03-lxd-cloud.md#3-cluster-the-machines-with-lxd-your-first-cloud)
