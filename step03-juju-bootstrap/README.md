# Model Driven Operated Micro Cloud with Juju

```sh
$ multipass shell juju
ubuntu@juju:~$ juju status
Model    Controller    Cloud/Region  Version  SLA          Timestamp
default  bare-default  bare/default  2.9.14   unsupported  18:54:40+02:00

App  Version  Status  Scale  Charm  Store  Channel  Rev  OS      Message
lxd           active      3  lxd    local             0  ubuntu  

Unit    Workload  Agent  Machine  Public address  Ports  Message
lxd/0*  active    idle   0        192.168.64.33          
lxd/1   active    idle   1        192.168.64.34          
lxd/2   active    idle   2        192.168.64.32          

Machine  State    DNS            Inst id               Series  AZ  Message
0        started  192.168.64.33  manual:192.168.64.33  focal       Manually provisioned machine
1        started  192.168.64.34  manual:192.168.64.34  focal       Manually provisioned machine
2        started  192.168.64.32  manual:192.168.64.32  focal       Manually provisioned machine
```

## Prepare the credentials to connect to our LXD micro cloud

The commands this first part are executed from the 'juju' controller machine and one of the micro cloud nodes machines.

First, let's get access to our LXD cloud locally.

```sh
$ multipass exec node1 -- lxc remote add microcloud 192.168.64.32
Generating a client certificate. This may take a minute...
Certificate fingerprint: abc123def
ok (y/n)? y
Admin password for microcloud: # exit with ctrl-c
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

Then use your favorite editor (e.g. `nano credentials.yaml`) to add the following yaml structure:
```yaml
credentials:
  microcloud:
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
$ multipass transfer node1:/home/ubuntu/snap/lxd/common/config/client.crt ./
$ multipass transfer ./client.crt juju:/home/ubuntu/client.crt
$ rm client.crt
$ multipass exec juju -- bash -c 'juju run-action lxd/0 add-trusted-client cert="$(cat /home/ubuntu/client.crt)" --wait'
```

Adding the remote should now work:
```sh
ubuntu@node1:~$ multipass shell node1
ubuntu@node1:~$ lxc remote add microcloud 192.168.64.32
ubuntu@node1:~$ lxc remote switch microcloud
```

We can now securely access our LXD cluster from the Juju controller:

```sh
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

## Install Juju on the micro cloud

```sh
ubuntu@node1:~$ sudo snap install juju --classic
ubuntu@node1:~$ juju add-cloud microcloud
```

## Register our LXD micro cloud as a Juju cloud
```sh
# Let's add our LXD cloud to Juju, naming it "microcloud"
ubuntu@node1:~$ juju add-cloud microcloud

Select cloud type: lxd

Enter the API endpoint url for the remote LXD server: https://192.168.64.32:8443

Enter region [default]: 

Enter the API endpoint url for the region [use cloud api url]: 

Enter another region? (y/N): 

Cloud "microcloud" successfully added to your local client.

# We then need to add the authentication information to administrate our LXD cloud
ubuntu@node1:~$ juju add-credential microcloud -f credentials.yaml
Credential "admin" added locally for cloud "microcloud".

# Now we can bootstrap our LXD cloud, asking Juju to install a controller on it
ubuntu@node1:~$ juju bootstrap microcloud
Creating Juju controller "microcloud-default" on microcloud/default
...
Bootstrap complete, controller "microcloud-default" is now available
Controller machines are in the "controller" model
Initial model "default" added

# Once the bootstrap is complete, the "status" command will show our cloud is properly registered
ubuntu@node1:~$ juju status
Model    Controller          Cloud/Region        Version  SLA          Timestamp
default  microcloud-default  microcloud/default  2.9.12   unsupported  13:05:13+02:00

Model "admin/default" is empty.
```

<!-- ## (Bonus) Control it from the outside

We are going to add our micro cloud Juju controller to our "Juju" controller machine, to control our micro cloud from the outside.

```sh
$ multipass shell node1
ubuntu@node1:~$ juju add-user juju
User "juju" added Please send this command to juju:
    juju register abc123def

# Granting the new user "juju" superadmin permissions over the default model and the 'microcloud' lxd cloud
ubuntu@node1:~$ juju grant juju admin default
ubuntu@node1:~$ juju grant juju superuser

# switch to the 'juju' machine
$ multipass shell juju
# add a route to the microcloud lxd cluster
ubuntu@juju:~$ sudo ip route add 240.0.0.0/8 via 192.168.64.34
# register the remote cloud
ubuntu@juju:~$ juju register abc123def
``` -->