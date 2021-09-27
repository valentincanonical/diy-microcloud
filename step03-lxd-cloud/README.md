# Build your LXD micro cloud!

<!-- ## Option 1: Automated deployment using Juju and Charms
## Option 2: Manual installation -->

_Expected duration: 15mn_

### Initiate the first node

```sh
# Log in to one of the nodes
$ multipass shell node1

# Init the LXD cluster setup, answering questions as below
ubuntu@node1:~$ sudo lxd init
# Say "Yes" for LXD clustering
Would you like to use LXD clustering? (yes/no) [default=no]: yes
What IP address or DNS name should be used to reach this node? [default=192.168.64.32]: 
Are you joining an existing cluster? (yes/no) [default=no]: 
What name should be used to identify this node in the cluster? [default=node1]:
# Enable password authentication - You'll need the password later, write it down
Setup password authentication on the cluster? (yes/no) [default=no]: yes
Trust password for new clients: 
Again: 
Do you want to configure a new local storage pool? (yes/no) [default=yes]: 
Name of the storage backend to use (btrfs, dir, lvm, zfs) [default=zfs]: 
Create a new ZFS pool? (yes/no) [default=yes]: 
Would you like to use an existing empty block device (e.g. a disk or partition)? (yes/no) [default=no]: 
# I secured more space than the default to make sure we have enough to experiment
Size in GB of the new loop device (1GB minimum) [default=5GB]: 7GB    
Do you want to configure a new remote storage pool? (yes/no) [default=no]: 
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to configure LXD to use an existing bridge or host interface? (yes/no) [default=no]: 
Would you like to create a new Fan overlay network? (yes/no) [default=yes]: 
What subnet should be used as the Fan underlay? [default=auto]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:

# done! you can verify the cluster is well configured with your first node
ubuntu@node1:~$ lxc cluster list
To start your first instance, try: lxc launch ubuntu:18.04

+-------+----------------------------+----------+--------+-------------------+--------------+
| NAME  |            URL             | DATABASE | STATE  |      MESSAGE      | ARCHITECTURE |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node1 | https://192.168.64.32:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+
```

### Join the other nodes to the LXD cluster

```sh
# From one of the configured nodes, generate a token to simplify the clustering operation
ubuntu@node1:~$ lxc cluster add node2
Member node2 join token:
eyJzZXJ2ZXJfxZWZkIn0= # saved some space here, the actual token will be much longer
ubuntu@node1:~$ logout

# Log in to an unconfigured node
bash-3.2$ multipass shell node2

# Run the LXD init command
ubuntu@node2:~$ sudo lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: yes
What IP address or DNS name should be used to reach this node? [default=192.168.64.33]: 
# This time, answer "Yes" when asked if you're joining an existing cluster
Are you joining an existing cluster? (yes/no) [default=no]: yes
Do you have a join token? (yes/no) [default=no]: yes
# Copy and paste the token obtained previously for this node
Please provide join token: eyJzZXJ2ZXJfxZWZkIn0=
All existing data is lost when joining a cluster, continue? (yes/no) [default=no] yes
Choose "size" property for storage pool "local": 7GB
Choose "source" property for storage pool "local": 
Choose "zfs.pool_name" property for storage pool "local": 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 

# The node has been added properly, you can verify by listing all cluster members
ubuntu@node2:~$ lxc cluster ls
To start your first instance, try: lxc launch ubuntu:18.04

+-------+----------------------------+----------+--------+-------------------+--------------+
| NAME  |            URL             | DATABASE | STATE  |      MESSAGE      | ARCHITECTURE |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node1 | https://192.168.64.32:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+
| node2 | https://192.168.64.33:8443 | YES      | ONLINE | Fully operational | x86_64       |
+-------+----------------------------+----------+--------+-------------------+--------------+

# Repeat the operation for the remaining node: node3 and node4
ubuntu@node2:~$ lxc cluster add node3

...

# done! you should see your four nodes clustered together
ubuntu@node4:~$ lxc cluster list
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

```

### (Optional) Trying the LXD cloud

_Expected duration: 5mn_

```sh
# Log in on any configured node
ubuntu@node4:~$ lxc cluster list
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

# Let's launch a linux container of Ubuntu 20.04 LTS named "demo"
ubuntu@node4:~$ lxc launch ubuntu:20.04 demo
Creating demo
Starting demo                               

# Listing the containers and machines running on the cluster, we see our "demo" container
ubuntu@node4:~$ lxc list
+------+---------+---------------------+------+-----------+-----------+----------+
| NAME |  STATE  |        IPV4         | IPV6 |   TYPE    | SNAPSHOTS | LOCATION |
+------+---------+---------------------+------+-----------+-----------+----------+
| demo | RUNNING | 240.32.0.144 (eth0) |      | CONTAINER | 0         | node1    |
+------+---------+---------------------+------+-----------+-----------+----------+

# We can shell into it to introspect the environment - done!
ubuntu@node4:~$ lxc shell demo
root@demo:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.3 LTS
Release:	20.04
Codename:	focal

```