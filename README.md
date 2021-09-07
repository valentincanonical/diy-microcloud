# Arm, Ubuntu, K8s: Build Your Own Cloud for Edge Computing

**Have you dreamt of having your own home cloud but found it too complex?     
Micro clouds enable anyone to build a lightweight cloud anywhere.**

## Abstract

Building micro clouds or edge clouds has vast potential.
The edge is where the real world happens.
Edge clusters bring your technologies to the consumer for more privacy and less latency.
So, where to begin? With Arm, duh, but what about after that?

This workshop gets you started.
For beginners, from install, set-up and config of an Ubuntu K8s cluster.
And experts, talking DevOps, AI use cases, and inferences at the edge, on something as simple and accessible as a Raspberry Pi or a Jetson Nano.

<!-- ToDo: Section about "what are micro clouds" / Goals -->

<!-- ToDo: Insert a diagram here -->

## Minimum Configuration

> To enable the most to follow this tutorial at home, we won't require any specific hardware - only your workstation will do it.
> Nonetheless, you will find secondary paths and alternative options all along, including instructions to use a cluster of Raspberry Pis.
> The following requirements apply if you're willing to follow the primary path, using virtualisation to emulate multiple small devices.

<!-- ToDo: Instructions to use RPis  -->

- 16GB RAM recommended (8GB min required);
- Min 50GB of storage left;
- [Multipass installed](https://multipass.run/) for your platform.

Before attending, please make sure you can use [Multipass](https://multipass.run/). The following commands should run without any specific configuration:

<!-- <details>
    <summary>
Click to expand the instructions.
    </summary> -->

```sh
    $ multipass launch --name iamatest --mem 8G --disk 10G
    Launched: iamatest 

    $ multipass list
    Name                    State             IPv4             Image
    iamatest                Running           192.168.64.71    Ubuntu 20.04 LTS

    $ multipass exec iamatest -- sudo snap install microk8s --classic
    microk8s (1.21/stable) v1.21.3 from Canonical✓ installed

    $ multipass exec iamatest -- sudo microk8s status --wait-ready
    microk8s is running
    high-availability: no
    datastore master nodes: 127.0.0.1:19001
    datastore standby nodes: none
    ...

    $ multipass stop iamatest
    $ multipass delete iamatest
    $ multipass list
    Name                    State             IPv4             Image
    iamatest                Deleted           --               Not Available

    $ multipass purge
```

> The sequence above should not take more than 10mn to run.
> Otherwise, please consider stopping any greedy processes, using a more powerful machine or a cloud-based virtual machine.

<!-- ToDo: test on a cloud virtual machine, and link to a tutorial. -->

<!-- </details> -->


## Building Your Home Lab Micro Cloud in 5 Steps

<!-- 

Options: 
- Virtual machines or Physical devices (RPis)
- Juju or not Juju
- Application 

 -->

To make it easier to follow, we split this tutorial into five steps with clear goals.
At the end of each step, a checkpoint will help you understand what is the outcome.
If you can't get to the checkpoint, please reach out for help to the staff or [online forums](ToDo: discourse).
<!-- ToDo: add proper forums + github issues -->

> **Checkpoint #0: [Minimum requirements matched.](#minimum-configuration)**    
> _If you are not going with the [Multipass](https://multipass.run/) (virtual machines) option, you can ignore this checkpoint as requirements will differ._

### #1 Prepare the bare metal nodes

_Expected duration: 5mn_

> In a usual micro cloud setup, this step - bare metal provisioning - would be fully automated. [Metal as a Service (MAAS)](https://maas.io/) can provision several to hundreds of physical servers and micro clouds spread over various locations. MAAS provides a way to flexibly deploy, manage, and maintain operating system loads on physical servers. It keeps track of all servers and their configurations available in the micro cloud. It is the base layer of the micro cloud stack.

> In this virtual and one-site configuration, we won't be using MAAS. As micro clouds are fully modular, this will allow us to focus on the virtualisation and K8s layers. I invite you to [read more about MAAS](https://maas.io/tutorials) to automate your bare metal provisioning in further micro cloud deployments!

This first step guides you through the provisioning of four Ubuntu machines using Multipass. The four virtual machines will emulate four physical nodes - let them be Raspberry Pis or any other small edge hardware.

```sh
# Let's use bash to loop over the creation of four Ubuntu machines using Multipass
$ for i in {1..4}; do multipass launch --name node$i --mem 4G --disk 10G; done;
Launched: node1                                                                 
Launched: node2                                                                 
Launched: node3                                                                 
Launched: node4

$ multipass list
Name                    State             IPv4             Image
node1                   Running           192.168.64.32    Ubuntu 20.04 LTS
node2                   Running           192.168.64.33    Ubuntu 20.04 LTS
node3                   Running           192.168.64.34    Ubuntu 20.04 LTS
node4                   Running           192.168.64.35    Ubuntu 20.04 LTS

$ multipass shell node1
ubuntu@node1:~$ # That's it! We now have four Ubuntu identical machines ready-to-go
```

If you are using Raspberry Pis instead, you can [follow this tutorial to install the latest Ubuntu Server](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview). I recommend using RPi 4+ 8GB, or above. (For stable deployments, I recommend you consider [the USB boot option](https://ubuntu.com/tutorials/how-to-install-ubuntu-desktop-on-raspberry-pi-4#4-optional-usb-boot) from an SSD instead of the SD card - from Ubuntu 20.10.)

> **Checkpoint #1: Four Ubuntu machines on the same network.**

### #2 Cluster the machines with LXD: your first cloud!

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #2: Four-node LXD cluster ready to operate.**

### #3 Register your cloud for model-driven operations

<!-- Optional (if not going for Juju) -->

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #3: LXD cloud can be controlled remotely.**

### #4 Create on-demand MicroK8s clusters: a micro cloud dream

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #4: MicroK8s cluster on LXD, up and running.**

### #5 Run cloud-native applications at the edge with micro clouds

<!-- Bonus -->

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #4: Application deployed on *production* MicroK8s.**

<!-- ToDo: find a better application use case for the micro cloud scenario. -->

<!-- Ideas for later: add distributed storage, add MAAS, add networking -->

### Authors/Reviewers

- Valentin Viennot, Product Manager, Canonical (Twitter: [@ValentinViennot](https://twitter.com/valentinviennot))
