## Mellanox OFED

Change into OFED installation directory:

```
cd ~/week3/mellanox
```

Install dependencies:

```
yum -y install numactl-libs atk tcsh tk python-devel rpm-build
```

Uncompress installation directory:

```
tar -zxvf MLNX_OFED_LINUX-5.0-1.0.0.0-rhel7.7-x86_64.tgz
```

Change into installation directory:

```
cd MLNX_OFED_LINUX-5.0-1.0.0.0-rhel7.7-x86_64
```

Install OFED (again, do not be alarmed if you do not receive any output for several minutes):

```
./mlnxofedinstall --skip-distro-check --add-kernel-support
```

Move installation directory to CHROOT environment:

```
mv ../MLNX_OFED_LINUX-5.0-1.0.0.0-rhel7.7-x86_64.tgz $CHROOT
```

Install dependencies on CHROOT environment:

```
yum -y --installroot=$CHROOT install numactl-libs gtk2 atk cairo gcc-gfortran tcsh libnl3 tcl tk python-devel make lsof redhat-rpm-config rpm-build createrepo
```

Mount directories on the master node to the CHROOT environment so OFED can be installed successfully:

```
mount --bind /dev $CHROOT/dev
mount --bind /sys $CHROOT/sys
mount --bind /proc $CHROOT/proc
```

Change into the CHROOT console (essentially, you will be executing commands that will be reflected upon the image being built for the compute nodes rather than the entire master node):

```
chroot $CHROOT
```

Uncompress installation files:

```
tar -zxvf MLNX_OFED_LINUX-5.0-1.0.0.0-rhel7.7-x86_64.tgz
```

Change into installation directory:

```
cd MLNX_OFED_LINUX-5.0-1.0.0.0-rhel7.7-x86_64
```

Install OFED:

```
./mlnxofedinstall --skip-distro-check --add-kernel-support --distro rhel7u7
```

Remove installation directory now that OFED has been installed on the CHROOT environment:

```
cd .. && rm -rf MLNX_OFED_LINUX-5.0-1.0.0.0*
```

Exit the CHROOT environment:

```
exit
```

Remove installation directory now that OFED has been installed on the master node:

```
cd ~/week3/mellanox && rm -rf MLNX_OFED_LINUX-5.0-1.0.0.0-rhel7.7-x86_64
```

Unmount the directories we mounted earlier as they are no longer needed in the CHROOT environment:

```
umount $CHROOT/proc
umount $CHROOT/sys
umount -l $CHROOT/dev
```

Rebuild the VNFS image, so the changes will be available on the compute nodes once they are rebooted:

```
wwvnfs --chroot=$CHROOT
```

From the previous exercises, here are instructions for rebooting the compute node:

```
ipmitool -H 10.2.2.2 -U USERID -P PASSW0RD chassis power cycle
```
