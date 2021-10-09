# Installing and Configuring OpenHPC

NOTE:
```
Be sure to first connect to [user]@cs74.stanford.edu
```

Connect to the master node (default password is stanford):
```
ssh [user]@hpcc-cluster-[C].stanford.edu
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

The following information is very important: the IP addresses and hostnames of the compute nodes will need to be assigned very carefully. For this class, we are following this pattern for the naming of the nodes: ```compute-[C]```.

For the IP address, it is defined this way: ```10.[C].2```. So, if you wanted to define the compute nodes for ```hpcc-cluster-1```, you would run these commands. Of course, remember to add the MAC addresses for each corresponding node. As a reminder, they can be found here.

Please make sure to replace ```MAC_ADDR``` with the correct address for the node you are adding (as well as the hostname and IP address)!
```
wwsh -y node new compute-1 --ipaddr=10.1.2 --hwaddr=MAC_ADDR
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

We used ipmitool a bit earlier, and we will need to use it again. Using part of the schema defined above, the ipmi addresses for the nodes will follow this formula: ```10.[C].3```. Notice the 3 at the end. So, to restart the compute nodes, you would run the following:

Hint: you need to run the command below once (once for your one compute node).
```
ipmitool -H 10.[C].3 -U USERID -P PASSW0RD chassis power cycle
```

For instance, the ipmi address for ```compute-1``` is ```10.1.3```.

You can view progress of the installation of the operating system onto compute nodes with the following command line option:
```
tail -f /var/log/messages
```

The compute nodes will go through a DHCP and PXE process, and end with a mount process. To quit watching the log, use ctrl+c.

To verify that the compute nodes have booted, you can ping their hostname, i.e:
```
ping compute-1
```

The output should resemble this:
```
PING compute-1-12.localdomain (10.1.12.2) 56(84) bytes of data.
64 bytes from compute-1.localdomain (10.1.2): icmp_seq=1 ttl=64 time=0.244 ms
64 bytes from compute-1.localdomain (10.1.2): icmp_seq=2 ttl=64 time=0.257 ms
64 bytes from compute-1.localdomain (10.1.2): icmp_seq=3 ttl=64 time=0.253 m
```


Add your and your partner(s) SUNet IDs to the system:
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
pdsh -w compute-[C] /warewulf/bin/wwgetfiles
```

To check if the Kerberos configuration worked, run the following commands (# denote comments, not commands you need to run):
```
# This command checks if the "useradd" comand was successful. Once you log into the user, immediately logout or click Ctrl-D.
su - [sunetid]
logout
kinit [sunetid]
klist
kdestroy
klist
```

The following commands can be used to check the uptime on the compute nodes:
```
pdsh -w compute-[C] uptime
# OR
wwsh ssh c* uptime
```

Output should resemble:
```
compute-1:  19:34:16 up  5:24,  0 users,  load average: 0.00, 0.01, 0.04
```

Important: remember when we used wwsh file import to sync files between the master and compute nodes? To resync them at any time, you can run the following command:
```
wwsh file resync
```

You might find yourself using this command if you make any changes to the Slurm configuration, add any users to the machine, or modify other files that you may have selected to sync.
