[< back to the introduction](./README.md#building-your-home-lab-micro-cloud-in-5-steps)

# #1 Prepare the bare metal nodes

_Expected duration: 10mn_

> In a usual micro cloud setup, this step - bare metal provisioning - would be fully automated. [Metal as a Service (MAAS)](https://maas.io/) can provision several to hundreds of physical servers and micro clouds spread over various locations. MAAS provides a way to flexibly deploy, manage, and maintain operating system loads on physical servers. It keeps track of all servers and their configurations available in the micro cloud. It is the base layer of the micro cloud stack.

> In this virtual and one-site configuration, we won't be using MAAS. As micro clouds are fully modular, this will allow us to focus on the virtualisation and K8s layers. We invite you to [read more about MAAS](https://maas.io/tutorials) to automate your bare metal provisioning in further micro cloud deployments!

This first step guides you through the provisioning of three Ubuntu machines using Multipass. The three virtual machines will emulate three physical nodes - let them be Raspberry Pis or any other small edge hardware. For a more resilient configuration, you might want to consider adding a fourth more node... it's as simple as repeating everything one more time!
<!-- ToDo: backlink to 1,2,3,4 section -->

```sh
#!/bin/bash

# Let's use bash to loop over the creation of four Ubuntu machines using Multipass
$ for i in {1..3}; do multipass launch --name node$i --mem 8G --disk 10G --cpus 4; done;
Launched: node1
Launched: node2
Launched: node3

$ multipass list
Name                    State             IPv4             Image
node1                   Running           192.168.64.32    Ubuntu 20.04 LTS
node2                   Running           192.168.64.33    Ubuntu 20.04 LTS
node3                   Running           192.168.64.34    Ubuntu 20.04 LTS

$ multipass shell node1
ubuntu@node1:~$ # That's it! We now have three Ubuntu machines ready-to-go
ubuntu@node1:~$ logout
```

If you are using Raspberry Pis instead, you can [follow this tutorial to install the latest Ubuntu Server](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview). We recommend using RPi 4+ 8GB, or above. (For stable deployments, we recommend you consider [the USB boot option](https://ubuntu.com/tutorials/how-to-install-ubuntu-desktop-on-raspberry-pi-4#4-optional-usb-boot) from an SSD instead of the SD card - from Ubuntu 20.10.)

---

> **Checkpoint #1: Three or four Ubuntu machines on the same network.**

<img alt="Four Ubuntu machines on the same network." src="./img/checkpoint-01.png" width="600" />

---

[Next step (2/5): Register for Model-Driven Operations >](./step-02-model-driven-operations.md#2-register-for-model-driven-operations)