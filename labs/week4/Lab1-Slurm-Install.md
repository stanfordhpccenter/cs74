Install Slurm:
```
yum -y install ohpc-slurm-server
```

Configure Slurm to add master node hostname:
```
perl -pi -e "s/ControlMachine=\S+/ControlMachine=hpcc-cluster-[C]/" /etc/slurm/slurm.conf
```

Change default node state for nodes returning to service (automatically return to services if all resources report correctly):
```
perl -pi -e "s/ReturnToService=1/ReturnToService=2/" /etc/slurm/slurm.conf
```

Now, we need to configure the hostnames inside the Slurm settings. Remember, the hostname for the compute node is: compute-[C], where [C] is the cluster number (i.e. 15).

This is what the end of ```/etc/slurm/slurm.conf``` will look like at this point:
```
NodeName=c[1-4] Sockets=2 CoresPerSocket=8 ThreadsPerCore=2 State=UNKNOWN
PartitionName=normal Nodes=c[1-4] Default=YES MaxTime=24:00:00 State=UP
ReturnToService=2
```

Go ahead and start a text editor (nano, vim, emacs) and start editing /etc/slurm/slurm.conf. Replace ```c[1-4]``` in the first line with your compute node hostname. Carrying on with the previous example, if our cluster name is ```hpcc-cluster-15```, the compute node scheme would be ```compute-15```.

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
NodeName=compute-[C] Sockets=2 CoresPerSocket=8 ThreadsPerCore=2 State=UNKNOWN
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

Add Slurm support to the compute nodes:
```
yum -y --installroot=$CHROOT install ohpc-slurm-client
```

Import Slurm and Munge files to Warewulf database
```
wwsh file import /etc/slurm/slurm.conf
wwsh file import /etc/munge/munge.key
```
Enable Slurm and Munge daemon in VNFS image:
```
chroot $CHROOT systemctl enable slurmd
chroot $CHROOT systemctl enable munge
```

Sync updated password and associated files following installation of Slurm client on VNFS image:
```
wwsh file resync passwd shadow group
```

Reassemble VNFS image
```
wwvnfs --chroot=$CHROOT
```

Update provisioning to include new files imported to database:
```
wwsh -y provision set "compute-*" --vnfs=centos7 --bootstrap=`uname -r` --files=dynamic_hosts,passwd,group,shadow,network,slurm.conf,munge.key
```

Reboot compute node
```
ssh compute-1-1 reboot
```

NOTE: If you see an error like the following, restart ```munge```:
```
[root@hpcc-cluster-[N] ~]# sinfo
sinfo: error: If munged is up, restart with --num-threads=10
sinfo: error: Munge encode failed: Failed to access "/var/run/munge/munge.socket.2": No such file or directory
```
