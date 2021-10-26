Install Node Health Check
```
yum -y install nhc-ohpc
yum -y --installroot=$CHROOT install nhc-ohpc
```

Modify /etc/nhc/nhc.conf similar to this:
```
# NHC Configuration File (sample)
#
### Filesystem checks
#
* || check_fs_mount_rw -t "tmpfs" -s "tmpfs" -f "/"
* || check_fs_mount_rw -t "proc" -s "proc" -f "/proc"
* || check_fs_mount_rw -t "sysfs" -s "sysfs" -f "/sys"
* || check_fs_mount_rw -t "devtmpfs" -s "devtmpfs" -f "/dev"
* || check_fs_mount_rw -t "nfs" -s "10.1.1.1:/opt" -f "/opt"
#
### Hardware checks
#
compute-* || check_hw_ib 56
largemem-* || check_hw_ib 56
* || check_hw_eth eth0
```

Import into database:
```
wwsh file import /etc/nhc/nhc.conf
```
