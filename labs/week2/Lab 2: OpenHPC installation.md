### Installing and Configuring OpenHPC

Connect to the master node (default password is stanford):
```
ssh root@me344-cluster-[C].stanford.edu
```

Disable SELinux:
```
perl -pi -e "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
```

Reboot the machine:
```
reboot
```

Once rebooted, connect through SSH and verify that SELinux is disabled:
```
sestatus
```

Disable the firewall service:
```
systemctl stop firewalld
systemctl disable firewalld
```

Add Kernel Packages
```
yum -y install http://vault.centos.org/7.7.1908/os/x86_64/Packages/kernel-debug-3.10.0-1062.el7.x86_64.rpm
yum -y install http://vault.centos.org/7.7.1908/os/x86_64/Packages/kernel-debug-devel-3.10.0-1062.el7.x86_64.rpm
```

Lock Kernel Version
```
yum -y install yum-plugin-versionlock 
yum versionlock *-3.10.0-1062.el7.x86_64
```

Install the repository for OpenHPC:
```
yum -y install http://build.openhpc.community/OpenHPC:/1.3/CentOS_7/x86_64/ohpc-release-1.3-1.el7.x86_64.rpm 
```

Install provisioning services:
```
yum -y install ohpc-base
yum -y install ohpc-warewulf
```

Configure time services:
```
systemctl enable ntpd.service
echo "server time.stanford.edu" >> /etc/ntp.conf
systemctl restart ntpd
```

Install Slurm:
```
yum -y install ohpc-slurm-server
```

Configure Slurm to add master node hostname:
```
perl -pi -e "s/ControlMachine=\S+/ControlMachine=me344-cluster-[C]/" /etc/slurm/slurm.conf
```

Change default node state for nodes returning to service (automatically return to services if all resources report correctly):
```
perl -pi -e "s/ReturnToService=1/ReturnToService=2/" /etc/slurm/slurm.conf
```

Now, we need to configure the hostnames inside the Slurm settings. Remember, the hostname scheme for the compute nodes is: compute-[C]-[12-14], where [C] is the cluster number (i.e. 15).

This is what the end of ```/etc/slurm/slurm.conf``` will look like at this point:
```
NodeName=c[1-4] Sockets=2 CoresPerSocket=8 ThreadsPerCore=2 State=UNKNOWN
PartitionName=normal Nodes=c[1-4] Default=YES MaxTime=24:00:00 State=UP
ReturnToService=2
```

Go ahead and start a text editor (nano, vim, emacs) and start editing /etc/slurm/slurm.conf. Replace ```c[1-4]``` in the first line with your compute node scheme. Carrying on with the previous example, if our cluster name is ```me344-cluster-15```, the compute node scheme would be ```compute-15-[12-14]```.

For the second line, you can set ```Nodes=ALL```. In addition to the hostname configuration, we need to let Slurm know the specifics about our hardware. Run the lscpu command on your master node. The output should resemble the following:

```
[root@me344-cluster-15 ~]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                32
On-line CPU(s) list:   0-31
Thread(s) per core:    2
Core(s) per socket:    8
Socket(s):             2
...
```

Note the following options in the Slurm configuration ```ThreadsPerCore```, ```CoresPerSocket```, ```Sockets```. If the values that Slurm displays resemble what is being outputted by the respective ```lscpu``` fields, then you are good to go. Otherwise, update the Slurm settings to match what ```lscpu``` outputs.

By the end, this is what the end of your Slurm configuration ```(/etc/slurm/slurm.conf)``` should look like:
```
NodeName=compute-[C]-[12-14] Sockets=2 CoresPerSocket=8 ThreadsPerCore=2 State=UNKNOWN
PartitionName=normal Nodes=ALL Default=YES MaxTime=24:00:00 State=UP
ReturnToService=2
```
Hint: ```Nodes=ALL``` is case insensitive — you must format it this way or it will not work.

Start and enable Slurm and Munge:
```
systemctl enable munge
systemctl start munge
systemctl enable slurmctld
systemctl start slurmctld
```

Configure Warewulf, our provisioning utility to use the local network ```(enp6s0f1)``` as the provisioning interface:
```
perl -pi -e "s/device = eth1/device = enp6s0f1/" /etc/warewulf/provision.conf
```

Enable tftp service for compute node image distribution:
```
perl -pi -e "s/^\s+disable\s+= yes/ disable = no/" /etc/xinetd.d/tftp
```

Restart and enable relevant services to support provisioning on the master node:
```
systemctl restart xinetd
systemctl enable mariadb.service
systemctl restart mariadb
systemctl enable httpd.service
systemctl restart httpd
systemctl enable dhcpd.service
```

Define the CHROOT environment location for the compute nodes:
```
export CHROOT=/opt/ohpc/admin/images/centos7
```

Add to your local environment for future use:
```
echo "export CHROOT=/opt/ohpc/admin/images/centos7" >> /root/.bashrc
```

Build initial CHROOT image:
```
wwmkchroot centos-7 $CHROOT
```

Add Kernel Packages
```
yum -y --installroot=$CHROOT install http://vault.centos.org/7.7.1908/os/x86_64/Packages/kernel-3.10.0-1062.el7.x86_64.rpm
yum -y --installroot=$CHROOT install http://vault.centos.org/7.7.1908/os/x86_64/Packages/kernel-headers-3.10.0-1062.el7.x86_64.rpm
yum -y --installroot=$CHROOT install http://vault.centos.org/7.7.1908/os/x86_64/Packages/kernel-devel-3.10.0-1062.el7.x86_64.rpm
yum -y --installroot=$CHROOT install http://vault.centos.org/7.7.1908/os/x86_64/Packages/kernel-debug-3.10.0-1062.el7.x86_64.rpm
yum -y --installroot=$CHROOT install http://vault.centos.org/7.7.1908/os/x86_64/Packages/kernel-debug-devel-3.10.0-1062.el7.x86_64.rpm
```

Lock Kernel Version
```
yum -y --installroot=$CHROOT install yum-plugin-versionlock 
chroot $CHROOT
yum versionlock *-3.10.0-1062.el7.x86_64
exit
```

Install the base package for compute nodes through OpenHPC:
```
yum -y --installroot=$CHROOT install ohpc-base-compute
```

Enable DNS on compute nodes:
```
cp -p /etc/resolv.conf $CHROOT/etc/resolv.conf
```

Add Slurm support to the compute nodes:
```
yum -y --installroot=$CHROOT install ohpc-slurm-client
```

Install time service:
```
yum -y --installroot=$CHROOT install ntp
```

Install modules user environment and ipmitool:
```
yum -y --installroot=$CHROOT install lmod-ohpc ipmitool
```

Initialize Warewulf database and SSH keys for the compute nodes:
```
wwinit database
wwinit ssh_keys
cat ~/.ssh/cluster.pub >> $CHROOT/root/.ssh/authorized_keys
```

Add NFS client mounts of ```/home``` and ```/opt/ohpc/pub``` to base image:
```
echo "10.1.1.1:/home /home nfs nfsvers=3,nodev,nosuid,noatime 0 0" >> $CHROOT/etc/fstab
echo "10.1.1.1:/opt/ohpc/pub /opt/ohpc/pub nfs nfsvers=3,nodev,noatime 0 0" >> $CHROOT/etc/fstab
```

Export ```/home``` and OpenHPC public packages from master server:
```
echo "/home *(rw,no_subtree_check,fsid=10,no_root_squash)" >> /etc/exports
echo "/opt/ohpc/pub *(ro,no_subtree_check,fsid=11)" >> /etc/exports
exportfs -a
systemctl restart nfs-server
systemctl enable nfs-server
```

Enable NTP time service on compute node and add master host as local NTP server:
```
chroot $CHROOT systemctl enable ntpd
echo "server 10.1.1.1" >> $CHROOT/etc/ntp.conf
```

Define compute node forwarding destination:
```
echo "*.* @10.1.1.1:514" >> $CHROOT/etc/rsyslog.conf
```

Disable most logging on compute nodes. Emergency and boot logs will remain on compute nodes.
```
perl -pi -e "s/^\*\.info/\\#\*\.info/" $CHROOT/etc/rsyslog.conf
perl -pi -e "s/^authpriv/\\#authpriv/" $CHROOT/etc/rsyslog.conf
perl -pi -e "s/^mail/\\#mail/" $CHROOT/etc/rsyslog.conf
perl -pi -e "s/^cron/\\#cron/" $CHROOT/etc/rsyslog.conf
perl -pi -e "s/^uucp/\\#uucp/" $CHROOT/etc/rsyslog.conf
```

Certain files need to be added Warewulf sync list in order to ensure functionality:
```
wwsh file import /etc/passwd
wwsh file import /etc/group
wwsh file import /etc/shadow
wwsh file import /etc/slurm/slurm.conf
wwsh file import /etc/munge/munge.key
```

Set provisioning interface as the default networking device:
```
echo "GATEWAYDEV=eth0" > /tmp/network.$$
echo "GATEWAY=10.1.1.1" >> /tmp/network.$$
wwsh -y file import /tmp/network.$$ --name network
wwsh -y file set network --path /etc/sysconfig/network --mode=0644 --uid=0
```

Build bootstrap device:
```
wwbootstrap `uname -r`
```

Assemble VNFS image:
```
wwvnfs --chroot $CHROOT
```
Extremely Important: you will need to run the above command every time you make changes to the CHROOT environment (i.e. adding new packages, modifying files). Your changes will not be reflected if you don't run this command and restart the compute nodes.

Now, we will be adding the compute nodes to the Warewulf database. Please refer to this page to obtain the IP and MAC addresses.

The following information is very important: the IP addresses and hostnames of the compute nodes will need to be assigned very carefully. For this class, we are following this pattern for the naming of the nodes: ```compute-[C]-[N]```, where ```[N]``` is the number of the compute node. This number will range between 12 - 14 in this case.

For the IP address, it is defined this way: ```10.[C].[N].2```. So, if you wanted to define the compute nodes for ```me344-cluster-1```, you would run these commands. Of course, remember to add the MAC addresses for each corresponding node. As a reminder, they can be found here.

Please make sure to replace ```MAC_ADDR``` with the correct address for the node you are adding (as well as the hostname and IP address)!
```
wwsh -y node new compute-1-12 --ipaddr=10.1.12.2 --hwaddr=MAC_ADDR
wwsh -y node new compute-1-13 --ipaddr=10.1.13.2 --hwaddr=MAC_ADDR
wwsh -y node new compute-1-14 --ipaddr=10.1.14.2 --hwaddr=MAC_ADDR
```

Now, we are going to define the provisioning image for the compute nodes:
```
wwsh -y provision set "compute-*" --vnfs=centos7 --bootstrap=`uname -r` --files=dynamic_hosts,passwd,group,shadow,network,slurm.conf,munge.key
```

Restart DHCP and update PXE:
```
systemctl restart dhcpd
wwsh pxe update
```

We used ipmitool a bit earlier, and we will need to use it again. Using part of the schema defined above, the ipmi addresses for the nodes will follow this formula: ```10.[C].[N].3```. Notice the 3 at the end. So, to restart the compute nodes, you would run the following:

Hint: you need to run the command below 3 times (once for each compute node).
```
ipmitool -H 10.[C].[N].3 -U USERID -P PASSW0RD chassis power cycle
```

For instance, the ipmi address for ```compute-1-12``` is ```10.1.12.3```.

You can view progress of the installation of the operating system onto compute nodes with the following command line option:
```
tail -f /var/log/messages
```

The compute nodes will go through a DHCP and PXE process, and end with a mount process. To quit watching the log, use ctrl+c.

To verify that the compute nodes have booted, you can ping their hostname, i.e:
```
ping compute-1-12
```

The output should resemble this:
```
PING compute-1-12.localdomain (10.1.12.2) 56(84) bytes of data.
64 bytes from compute-1-12.localdomain (10.1.12.2): icmp_seq=1 ttl=64 time=0.244 ms
64 bytes from compute-1-12.localdomain (10.1.12.2): icmp_seq=2 ttl=64 time=0.257 ms
64 bytes from compute-1-12.localdomain (10.1.12.2): icmp_seq=3 ttl=64 time=0.253 m
```

Kerberos Authentication

Kerberos is an authentication method that the HPCC uses, allowing cluster users to sign in to machines using their SUNet IDs. First, install Kerberos on your master node, and copy the configuration file to the me344-cluster machine:

When specifying your SUNet ID, do not use your email address, only the ID.
```
yum -y install pam_krb5
scp [sunetid]@me344-cluster:/etc/krb5.conf /etc/
authconfig --enablekrb5 --update
```

Now, add your and your partner(s) SUNet IDs to the system:
```
useradd -m [sunetid]
useradd -m [partner_sunetid]
```

Push file changes to the compute nodes:
```
wwsh file resync passwd shadow group
```

If the above command does not work, using pdsh is a fall-back solution:
```
pdsh -w compute-[C]-[12-14] /warewulf/bin/wwgetfiles
```

To check if the Kerberos configuration worked, run the following commands (# denote comments, not commands you need to run):

# This command checks if the "useradd" comand was successful. Once you log into the user, immediately logout or click Ctrl-D.
```
su - [sunetid]
logout
kinit [sunetid]
klist
kdestroy
klist
```

The following commands can be used to check the uptime on the compute nodes:
```
pdsh -w compute-[C]-[12-14] uptime
```
# OR
```
wwsh ssh c* uptime
```

Output should resemble:
```
compute-1-14:  19:34:16 up  5:24,  0 users,  load average: 0.00, 0.01, 0.04
compute-1-13:  19:34:16 up  5:24,  0 users,  load average: 0.00, 0.01, 0.04
compute-1-12:  19:34:16 up  5:24,  0 users,  load average: 0.00, 0.01, 0.04
```

Important: remember when we used wwsh file import to sync files between the master and compute nodes? To resync them at any time, you can run the following command:
```
wwsh file resync
```

You might find yourself using this command if you make any changes to the Slurm configuration, add any users to the machine, or modify other files that you may have selected to sync.
