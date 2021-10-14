[< back to Step 4: Create on-demand MicroK8s clusters](./step-04-microk8s-cluster.md#4-create-on-demand-microk8s-clusters)

# #5 Run cloud-native applications at the edge with micro clouds

Your micro cloud is now ready, registered, and you know how to create lightweight MicroK8s clusters on demand. The next steps are optional, feel free to pick what's the most interesting to you. The goal there is to get you productive with fancy applications running on your homelab micro cloud.

<img alt="Micro cloud stack with kubernetes workloads running at the edge." src="./img/checkpoint-05.png" width="600" />

## Register your MicroK8s edge clusters with Portainer

_Expected duration: 10mn_

Portainer allows you to “build, manage and deploy containers in your Kubernetes environment quickly and easily. No more CLI, no more YAML, no more Kubernetes manifests. Just simple, fast, K8s configuration in a graphical UI, built on a trusted open source platform.”

The steps for [getting started with Portainer on MicroK8s](https://www.portainer.io/blog/how-to-deploy-portainer-on-microk8s) are fairly easy:

```sh
# If you used Juju to deploy MicroK8s
$ juju ssh microk8s/0
# Otherwise -- multipass shell node1; lxc shell worker1;

# **Enable the Portainer addon**
root@worker1:~$ microk8s enable portainer
```

Last step before accessing your Portainer dashboard: route the traffic from your machine to inside your micro cloud cluster.

You can either:
- A/ configure a route on your machine `ip route add <lxd-cluster-subnet> via <node1-ip>`
- B/ configure a reverse proxy on your micro cloud

We'll go with the reverse proxy option as it doesn't depend on your host configuration:

```sh
ubuntu@node1:~$ sudo apt update && sudo apt install -y nginx
# get the IP with juju status or lxc ls
ubuntu@node1:~$ IP_MICROK8S_CLUSTER=<REPLACE-WITH-MICROK8S-IP>
ubuntu@node1:~$ cat > /tmp/portainer.conf <<EOF
server { 
  listen 30777;
  location / {
          proxy_pass http://$IP_MICROK8S_CLUSTER:30777;
  }
}
EOF
ubuntu@node1:~$ sudo mv /tmp/portainer.conf /etc/nginx/sites-enabled/
ubuntu@node1:~$ sudo systemctl restart nginx
```

- Check the IP address of `node1` and access the Portainer dashboard at `http://<IP-address-node1>:30777` from your machine (eg; http://192.168.64.32:30777)

Now, as this is going to be an edge cluster, remotely managed, you probably want to register it to a central control place. With Portainer, this is super easy! Click [here and follow the instructions](https://documentation.portainer.io/v2.0/endpoints/edge/).

If you're just trying this out and don't have a "central control plane", this is fine. You can create a new MicroK8s cluster that you will register from the Portainer dashboard that you just deployed, just for fun! Refer to [section 4](./step-04-microk8s-cluster.md#4-create-on-demand-microk8s-clusters) on how to do that.

<!-- TODO: for the demo, use New App -> ubuntu/grafana -> port 30000 -> cp nginx/portainer.conf -->

## Beyond the workshop

- [Deploy an application on your kubernetes cluster from Canonical's LTS Docker Images](TODO)
- [Register your MicroK8s edge cluster with Juju](TODO)
- [Deploy applications to your micro cloud with Juju and Charmed Operators](TODO)
- [Register an edge endpoint with Portainer](https://documentation.portainer.io/v2.0/endpoints/edge/)
- Create more isolated on-demand kubernetes clusters on your edge micro cloud with    
  `juju add-model my-kubernetes && juju deploy microk8s -n1`

<!-- ## Register your MicroK8s edge clusters with Juju

_Expected duration: 10mn_

TODO TODO TODO

## Deploy applications to your micro cloud with Juju and Charms

_Expected duration: 10mn_

You first need to register your MicroK8s edge cluster with Juju. Click [here to scroll up](#register-your-microk8s-edge-clusters-with-juju).

TODO TODO TODO

-->

# How to clean your machine

If you used the Multipass option, it's super easy to clean things behind yourself once you're done with the tutorial:

```sh
$ multipass list 
Name                    State             IPv4             Image
juju                    Running           192.168.64.4     Ubuntu 20.04 LTS
node1                   Running           192.168.64.32    Ubuntu 20.04 LTS
node2                   Running           192.168.64.33    Ubuntu 20.04 LTS
node3                   Running           192.168.64.34    Ubuntu 20.04 LTS

# If you were already using Multipass on your machine,
# please make sure to replace '--all' with only the instances you wish to remove
$ multipass delete --all --purge

$ multipass list 
No instances found.

# done!
```

If you used the AWS cloud machines option, simply:
- "Terminate" the instances you launched for the tutorial
- Remove the microcloud key from "Network & Security" > "Key Pairs"
- Remove the microcloud Security Group from "Network & Security" > "Security Groups"

# Authors/Reviewers

- Valentin Viennot, Product Manager, Canonical (Twitter: [@ValentinViennot](https://twitter.com/valentinviennot))
- Pedro Cruz, Product Lead, Canonical (Twitter: [@ArduinoPedro](https://twitter.com/ArduinoPedro))

<!-- TODO: make review by Theodora -->

<!-- ToDo: get relevant reviews from the different product teams involved. -->

<!-- ToDo: validate license terms -->

Table of Content generated with [gh-md-toc](https://github.com/ekalinin/github-markdown-toc).

---

 Copyright 2021 Canonical Ltd.

   [Licensed under the Apache License](./LICENSE), Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.


[< back to the Introduction](./README.md#arm-ubuntu-k8s-build-your-own-cloud-for-edge-computing)
