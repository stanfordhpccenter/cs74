Install Node Health Check
```
yum -y install nhc-ohpc
yum -y --installroot=$CHROOT install nhc-ohpc
```

Reassemble VNFS image
```
wwvnfs --chroot=$CHROOT
```

Modify /etc/nhc/nhc.conf similar to this:
```
# NHC Configuration File (sample)
#
### Filesystem checks
#
* || check_fs_mount_rw -t "nfs" -s "10.1.1.1:/opt/ohpc/pub" -f "/opt/ohpc/pub"
#
### Hardware checks
#
compute-* || check_hw_ib 56
```

Import into database:
```
wwsh file import /etc/nhc/nhc.conf
```

Update provisioning to include new files imported to database:
```
wwsh -y provision set "compute-*" --vnfs=centos7 --bootstrap=`uname -r` --files=dynamic_hosts,passwd,group,shadow,network,slurm.conf,munge.key,nhc.conf
```

Reboot compute node:
```
ssh compute-1-1 reboot
```
