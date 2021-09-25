# Model Driven Operated Micro Cloud with Juju

## Install Juju

You can perform this step on any machine, even one that is not part of the cluster (but that can reach to your micro cloud nodes over the network). To avoid a

```sh
$ multipass shell node1

# we're using --classic as the Juju snap isn't using strict confinement
ubuntu@node1:~$ sudo snap install juju --classic
juju 2.9.12 from Canonical✓ installed
```

## Register our private LXD cloud as a Juju cloud

```sh
# We are going to need the cluster URL of one of the nodes for the next step
ubuntu@node1:~$ lxc cluster ls
+-------+----------------------------+----------+--------+-------------------+--------------+
| NAME  |            URL             | DATABASE | STATE  |      MESSAGE      | ARCHITECTURE |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node1 | https://192.168.64.32:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node2 | https://192.168.64.33:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node3 | https://192.168.64.34:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node4 | https://192.168.64.35:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+

# Let's add our LXD cloud to Juju, naming it "microcloud"
ubuntu@node1:~$ juju add-cloud microcloud
No current controller was detected and there are no registered controllers on this client: either bootstrap one or register one.
Cloud Types
  lxd
  maas
  manual
  openstack
  vsphere

Select cloud type: lxd

# Paste the URL of any of the nodes from the `lxc cluster list` command
Enter the API endpoint url for the remote LXD server: https://192.168.64.32:8443

Auth Types
  certificate

Enter region [default]: 

Enter the API endpoint url for the region [use cloud api url]: 

Enter another region? (y/N): 

Cloud "microcloud" successfully added to your local client.

# We then need to add the authentication information to administrate our LXD cloud
ubuntu@node1:~$ juju add-credential microcloud
This operation can be applied to both a copy on this client and to the one on a controller.
No current controller was detected and there are no registered controllers on this client: either bootstrap one or register one.
Enter credential name: lxd-credentials

Regions
  default

Select region [any region, credential is not region specific]: 

Auth Types
  certificate
  interactive

# Select the "interactive" mode as we used a password during the "lxd init" setup
Select auth type [interactive]: 

# Enter the password you used to configure your cluster at the "lxd init" step earlier
Enter trust-password: 

Generating client cert/key in "/home/ubuntu/.local/share/juju/lxd"
Uploaded certificate to LXD server.
Credential "lxd-credentials" added locally for cloud "microcloud".

# Now we can bootstrap our LXD cloud, asking Juju to install a controller on it
ubuntu@node1:~$ juju bootstrap microcloud
Creating Juju controller "microcloud-default" on microcloud/default
Looking for packaged Juju agent version 2.9.12 for amd64
Located Juju agent version 2.9.12-ubuntu-amd64 at https://streams.canonical.com/juju/tools/agent/2.9.12/juju-2.9.12-ubuntu-amd64.tgz
To configure your system to better support LXD containers, please see: https://github.com/lxc/lxd/blob/master/doc/production-setup.md
Launching controller instance(s) on microcloud/default...
 - juju-699bb9-0 (arch=amd64)          
Installing Juju agent on bootstrap instance
Fetching Juju Dashboard 0.8.1
Waiting for address
Attempting to connect to 240.32.0.103:22
Connected to 240.32.0.103
Running machine configuration script...
Bootstrap agent now started
Contacting Juju controller at 240.32.0.103 to verify accessibility...

Bootstrap complete, controller "microcloud-default" is now available
Controller machines are in the "controller" model
Initial model "default" added

# Once the bootstrap is complete, the "status" command will show our cloud is properly registered
ubuntu@node1:~$ juju status
Model    Controller          Cloud/Region        Version  SLA          Timestamp
default  microcloud-default  microcloud/default  2.9.12   unsupported  13:05:13+02:00

Model "admin/default" is empty.
```