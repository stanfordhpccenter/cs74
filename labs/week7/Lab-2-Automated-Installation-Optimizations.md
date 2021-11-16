## Guide to Automated Cluster Configuration

Please remember to carry out this assignment on your secondary cluster. If you don't have one, message Steve to assign you one.

[C] is your cluster number. For example, if you are configuring cluster 10, replace [C] with 10. 

0. Remember to apply the correct configurations to your master node that are needed prior to the following steps. You should reference the lab from week1.

1. Connect to your master node (default password is `stanford`):
```
ssh root@hpcc-cluster-[C].stanford.edu
```

2. Disable SELinux:
```
perl -pi -e "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
```

3. Reboot the machine:

Note: Your connection to the cluster will end, and you'll need to wait a few minutes while the cluster reboots before you can re-connect. 
```
reboot
```

4. Once rebooted, connect through SSH and verify that SELinux is disabled:
```
ssh root@hpcc-cluster-[C]

sestatus
```

5. Retrieve the recipe script:
```
cd /
wget https://raw.githubusercontent.com/davidrbradshaw/HPCC/master/CS74/recipe.sh
```

6. Retrieve the input.local file:
```
wget https://raw.githubusercontent.com/davidrbradshaw/HPCC/master/CS74/input.local
```

7. Revise the input file to suit your system's settings:
```
nano input.local
```

8. Add Infiniband, Charliecloud, and Singularity to the automated installation file recipe.sh

Add the following to the master node section following ```"yum -y install ohpc-warewulf"```
```
yum -y install singularity-ohpc charliecloud-ohpc
yum -y groupinstall "InfiniBand Support"
yum -y install opensm
systemctl enable opensm
```

Add the following to the compute node section following ```"wwsh file import /etc/munge/munge.key"```
```
yum -y --installroot=$CHROOT groupinstall "InfiniBand Support"
```

9. Use environment variable to define local input file:
```
export OHPC_INPUT_LOCAL=./input.local
```

10. Open access to the installation file:
```
chmod u+r+x recipe.sh
```

11. Run the local installation, which will take a while. Alternatively, you can use nohup to ensure the installation doesn't terminate if your session accidentally ends ```nohup ./recipe.sh &```:
```
./recipe.sh
```

12. Run this command for the compute node. If pinging doesn't work after a few minutes, run this command again.
```
ipmitool -H 10.2.2.2 -U USERID -P PASSW0RD chassis power cycle
```

13. To verify that the compute nodes have booted, you can ping their hostname, i.e:
```ping compute-1-1```

The output should resemble this:
```
PING compute-1-12.localdomain (10.1.12.2) 56(84) bytes of data.
64 bytes from compute-1-1.localdomain (10.1.12.2): icmp_seq=1 ttl=64 time=0.244 ms
64 bytes from compute-1-1.localdomain (10.1.12.2): icmp_seq=2 ttl=64 time=0.257 ms
64 bytes from compute-1-1.localdomain (10.1.12.2): icmp_seq=3 ttl=64 time=0.253 ms
```
