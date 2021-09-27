# Register your bare metal cloud with Juju

All the commands in this section are executed from the host (your machine). If you are using physical machines, for example Raspberry Pis, you will need to adapt the commands accordingly.

## Prereq

If you followed the [previous steps](../README.md#building-your-home-lab-micro-cloud-in-5-steps), you now are in a situation where you have at least three virtual or physical machines dedicated as micro cloud nodes and a controller machine for Juju.

```sh
$ multipass list
Name                    State             IPv4             Image
juju                    Running           192.168.64.4     Ubuntu 20.04 LTS
node1                   Running           192.168.64.32    Ubuntu 20.04 LTS
node2                   Running           192.168.64.33    Ubuntu 20.04 LTS
node3                   Running           192.168.64.34    Ubuntu 20.04 LTS
```

## Install Juju

The command to install [Juju](https://juju.is/) is pretty straightforward, and shouldn't take long!

```sh
multipass exec juju -- sudo snap install juju --classic
```

## Configure SSH access

You'll need SSH access from the Juju controller machine to your micro cloud nodes in order to operate them with Juju.
Let's generate an SSH key pair on the "Juju" machine and add the public key as an authorized host to the micro cloud nodes.

```sh
# Generate a new SSH key pair (accept all the default parameters)
multipass exec juju -- ssh-keygen
# Allow the Juju controller to ssh-login into itself
multipass exec juju -- bash -c 'echo "$(cat .ssh/id_rsa.pub)" >> .ssh/authorized_keys'
# Allow the Juju controller to ssh-login into other micro cloud nodes
SSHJUJU="$(multipass exec juju -- cat /home/ubuntu/.ssh/id_rsa.pub)"
for i in {1..3}; do multipass exec node$i -- bash -c "echo $SSHJUJU >> /home/ubuntu/.ssh/authorized_keys"; done;
```

## Create bare cloud and add micro cloud nodes

Now that we have everything in place, let's create a Juju bare cloud and add our physical/virtual machines to it.

```sh
$ multipass list
Name                    State             IPv4             Image
juju                    Running           192.168.64.4     Ubuntu 20.04 LTS
node1                   Running           192.168.64.32    Ubuntu 20.04 LTS
node2                   Running           192.168.64.33    Ubuntu 20.04 LTS
node3                   Running           192.168.64.34    Ubuntu 20.04 LTS

$ multipass exec juju -- juju add-cloud bare
This operation can be applied to both a copy on this client and to the one on a controller.
No current controller was detected and there are no registered controllers on this client: either bootstrap one or register one.
Cloud Types
  lxd
  maas
  manual
  openstack
  vsphere

# Select the "manual" cloud type
Select cloud type: manual

# Build the SSH connection string from the IP addresses retrieved from "multipass list"
Enter the ssh connection string for controller, username@<hostname or IP> or <hostname or IP>: ubuntu@192.168.64.4

Cloud "bare" successfully added to your local client.

# Bootstrap your cloud! This will install the Juju controller software to your machine
$ multipass exec juju -- juju bootstrap bare
Creating Juju controller "bare-default" on bare/default
Looking for packaged Juju agent version 2.9.14
Located Juju agent version 2.9.14-ubuntu at https://streams.canonical.com/juju/tools/agent/2.9.14/juju-2.9.14-ubuntu.tgz
Installing Juju agent on bootstrap instance
Fetching Juju Dashboard 0.8.1
Running machine configuration script...
Bootstrap agent now started
Contacting Juju controller at 192.168.64.4 to verify accessibility...
Bootstrap complete, controller "bare-default" is now available
Controller machines are in the "controller" model
Initial model "default" added

# Add your micro cloud nodes as machines to the bare cloud
$ multipass exec juju -- juju add-machine ssh:ubuntu@192.168.64.32
created machine 0

$ multipass exec juju -- juju add-machine ssh:ubuntu@192.168.64.33
created machine 1

$ multipass exec juju -- juju add-machine ssh:ubuntu@192.168.64.34
created machine 2

# ...done!

$ multipass exec juju -- juju status                                                          
Model    Controller    Cloud/Region  Version  SLA          Timestamp
default  bare-default  bare/default  2.9.14   unsupported  16:58:36+02:00

Machine  State    DNS            Inst id               Series  AZ  Message
0        started  192.168.64.32  manual:192.168.64.32  focal       Manually provisioned machine
1        started  192.168.64.33  manual:192.168.64.33  focal       Manually provisioned machine
2        started  192.168.64.34  manual:192.168.64.34  focal       Manually provisioned machine
```

<!-- for i in {2..4};do ssh ubuntu@192.168.64.3$i -- sudo /sbin/remove-juju-services; done; -->