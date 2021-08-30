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

<!-- ToDo: text to explain the checkpoints/parts -->

> **Checkpoint #0: Minimum requirements & Multipass installed.**

### #1 Set-up the four-node cluster

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #1: Four Ubuntu machines on the same network.**

### #2 Cluster the machines with LXD: your first cloud!

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #2: Four-node LXD cluster ready to operate.**

### #3 Register your cloud for model-driven operations

<!-- Optional (if not going for Juju) -->

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #3: LXD cloud can be controlled remotely.**

### #4 On demand MicroK8s clusters: a micro cloud dream

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #4: MicroK8s cluster on LXD, up and running.**

### #5 Cloud-native applications running at the edge on the micro cloud

<!-- Bonus -->

<!-- ToDo: Summary of the section's goals and outcomes -->

> **Checkpoint #4: Application deployed on *production* MicroK8s.**

<!-- ToDo: find a better application use case for the micro cloud scenario. -->
