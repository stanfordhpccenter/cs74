# Network File System (NFS) Mount Configuration
NOTE:
```
Be sure to first connect to [user]@cs74.stanford.edu
```
Connect to the master node (default password is stanford):
```
ssh [user]@hpcc-cluster-[C].stanford.edu
```
Add NFS client mounts of /home and /opt/ohpc/pub to base image:
```
echo "10.1.1.1:/home /home nfs nfsvers=3,nodev,nosuid,noatime 0 0" >> $CHROOT/etc/fstab
echo "10.1.1.1:/opt/ohpc/pub /opt/ohpc/pub nfs nfsvers=3,nodev,noatime 0 0" >> $CHROOT/etc/fstab
```

Export /home and OpenHPC public packages from master server:
```
echo "/home *(rw,no_subtree_check,fsid=10,no_root_squash)" >> /etc/exports
echo "/opt/ohpc/pub *(ro,no_subtree_check,fsid=11)" >> /etc/exports
exportfs -a
systemctl restart nfs-server
systemctl enable nfs-server
```
Reassemble VNFS image
```
wwvnfs --chroot $CHROOT
```

Restart the compute node using warewulf shell:
```
wwsh ssh compute-* reboot
```

You can view progress of the installation of the operating system onto compute nodes with the following command line option:
```
tail -f /var/log/messages
```

The compute node will go through a DHCP and PXE process, and end with a mount process similar to the following:
```
hpcc-cluster-18 rpc.mountd[13386]: authenticated mount request from 10.10.1.1:794 for /home (/home)
hpcc-cluster-18 rpc.mountd[13386]: authenticated mount request from 10.10.1.1:866 for /opt/ohpc/pub (/opt/ohpc/pub)
```

To quit watching the log, use ctrl+c.

To verify that the compute node has booted, you can ping their hostname, i.e:
```
ping compute-1-1
```

The output should resemble this:
```
PING compute-1-1.localdomain (10.10.1.1) 56(84) bytes of data.
64 bytes from compute-1-1.localdomain (10.10.1.1): icmp_seq=1 ttl=64 time=0.244 ms
64 bytes from compute-1-1.localdomain (10.10.1.1): icmp_seq=2 ttl=64 time=0.257 ms
64 bytes from compute-1-1.localdomain (10.10.1.1): icmp_seq=3 ttl=64 time=0.253 m
```
You should be able to SSH to the compute node at this point:
```
ssh compute-1-1
or
ssh [user]@compute-1-1
```
