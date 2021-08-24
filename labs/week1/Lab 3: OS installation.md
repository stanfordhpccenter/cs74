## Setting Up Your Cluster

This guide will outline the installation process for the cluster you will be working on for ME344. Make sure to only use the cluster assigned to you.

An important note about this guide: if you do not follow the instructions, step-by-step, there is a very high chance that you will run into issues. Don't worry about it too much, as this class is about learning how HPC works and making mistakes is a natural part of the process.

Also, if you simply copy-paste these commands into your terminal without understanding the underlying concepts, it will be difficult to debug issues with your cluster once you arrive to that point. Put in the effort to understand these concepts, and you will be successful.

Once assigned your cluster, be sure to note the entire hostname and especially the number, as it will be extremely important throughout this guide. When you see [C], it will be referring to the number of the cluster. For example, 15 corresponds to me344-cluster-15.

Important: Your assigned cluster is made up of 4 computers: one master node and three compute nodes. Continuing with the previous example, if you are assigned me344-cluster-15, the hostname of the master node is me344-cluster-15, and the compute nodes have a numerically assigned identifier from 12 - 14.

You will be setting the hostnames of the compute nodes later on in the guide, but here are the names of the compute nodes for the cluster me344-cluster-15:

1. ```compute-15-12```
2. ```compute-15-13```
3. ```compute-15-14```

The only number that will change between your cluster and the one used in this example is [C], which is 15 in this case but can range from 1 - 15. Pay close attention to the details of your cluster assignment!

You will need to be connected to the Stanford VPN in order to connect to any of our compute resources. Instructions about setting it up are available here.

### Week #1

#### Installing CentOS on the Master Node

Because you won't have physical access to the computers this quarter, we are going to follow a bit of a different process to install the operating system. You'll need to initialize the master node for your cluster and set it up for network boot, which will install the OS onto the machine.

Log into me344-cluster in order to execute ipmitool commands:

```
ssh [sunetid]@me344-cluster.stanford.edu
```

Set the next boot to PXE:

```
ipmitool -H me344-cluster-[C]-ipmi -U USERID -P PASSW0RD chassis bootdev pxe
```
Reboot the machine:

```ipmitool -H me344-cluster-[C]-ipmi -U USERID -P PASSW0RD chassis power cycle```

Due to the nature of the installation process, CS74 teaching staff will need to initiate the install between these steps. Hopefully you will only need to do this once, but contact us through if you need it to be initialized again.

Because we don't have physical access to the machine, we will need to verify the installation in another manner: using Serial over LAN (SOL) through IPMI. Serial is a type of communication interface that allows one to see what is happening (usually only text) on a computer, rather than using HDMI/VGA or other types of visual connectors. SOL allows us to connect to see what is happening on a screen through a network interface, in this case, IPMI.

Now, let's connect to the machine through SOL so you can see what is happening on the console.

ipmitool -H me344-cluster-[C]-ipmi -U USERID -P PASSW0RD -I lanplus sol activate
Once you reach a screen that says PXE Boot, the process is working so far. The entire installation process from this point will take between 5 - 10 minutes. You will know that the OS is installed when the screen says that it is booting into CentOS. You will not receive any output after this point.

If nothing is happening on the screen, the operating system may not have been provisioned to this specific machine. Please reach out to course staff and they will provision it.

In a new terminal window, we need to disconnect the SOL session. SSH back into me344-cluster, run the deactivate command, and close the terminal windows that you used to connect to me344-cluster:

ssh [sunetid]@me344-cluster.stanford.edu
...
[sunetid@me344-cluster ~]# ipmitool -H me344-cluster-[C]-ipmi -U USERID -P PASSW0RD -I lanplus sol deactivate
Configure Master Node
Terminate the current SSH session, and wait a few minutes for the master node to boot up.

Now, connect to the master node (default password is stanford):

ssh root@me344-cluster-[C].stanford.edu
The first thing to verify is the state of the networks. Your master node will have two of them: an "internal" network, and an external network, also known as the Internet. Each network has an interface associated with it. The interface for the internal network is named enp6s0f1 and the name of the external interface is eno1.

To verify the state of the networks, run ifconfig. You should see that the interfaces eno1 and enp6s0f1 both have IP addresses and are connected to the network.

Now, modify /etc/hosts so the machine knows its hostname. For this step, you will need to know [NNN], the last octet of your public IP address. It can be obtained by running ifconfig and looking at the entry for eno1.

echo "171.64.116.[NNN] me344-cluster-[C].stanford.edu" >> /etc/hosts
echo "10.1.1.1 me344-cluster-[C].localdomain me344-cluster-[C]" >> /etc/hosts
If you have completed these steps, you are now done with Week #1 tasks. If your partner has not completed their assignment yet, please notify them so they can start as soon as possible. Please message course staff on Canvas if you need help provisioning the operating system.
