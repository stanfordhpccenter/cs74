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
./mlnxofedinstall --skip-distro-check --add-kernel-support
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
cd .. && rm -rf MLNX_OFED_LINUX-5.0-1.0.0.0
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

From the previous week, here are instructions for rebooting the compute nodes:

> We used ipmitool a bit earlier, and we will need to use it again. Using part of the schema defined above, the ipmi addresses for the nodes will follow this formula: 10.[C].[N].3. Notice the 3 at the end. So, to restart the compute nodes, you would run the following:

> Hint: you need to run the command below 3 times (once for each compute node).

```
ipmitool -H 10.[C].[N].3 -U USERID -P PASSW0RD chassis power cycle
```

# Mellanox HPC-X

Install dependency:

```
yum -y install numactl-devel
```

Change into installation directory:

```
cd ~/week3/mellanox
```

Uncompress installation files:

```
tar -jxvf *.tbz && mv hpcx-v2.7.0-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64 /opt/ohpc/pub/apps/hpcx-v2.8.1
```

Load compilation modules on your master node:

```
module load gnu8 intel
```

Save HPC-X directory in temporary variable and change into directory:

```
export HPCX_DIR=/opt/ohpc/pub/apps/hpcx-v2.8.1 && cd $HPCX_DIR
```

Move installation files to appropriate location:

```
mv ompi{,.orig}
cd sources
```

Uncompress OpenMPI installation files and change into directory:

```
tar -zxvf openmpi-gitclone.tar.gz
cd openmpi-gitclone
```

Configure compiler for installation:

```
CC=icc CXX=icpc FC=ifort F77=ifort ./configure --prefix=$HPCX_DIR/ompi --with-libevent=internal --enable-mpi1-compatibility --without-xpmem --without-cuda --with-slurm --with-platform=contrib/platform/mellanox/optimized --with-hcoll=$HPCX_DIR/hcoll --with-ucx=$HPCX_DIR/ucx 2>&1 | tee configure.log
```

Compile software:

```
make -j16 install 2>&1 | tee make.log
```

Finalize installation by creating module for later use:

```
cd $HPCX_DIR
cp -ar ompi.orig/tests ompi
cp ~/week3/mellanox/hpcx /opt/ohpc/pub/modulefiles/
cp ~/week3/intel-psxe/psxe-2020 /opt/ohpc/pub/modulefiles/intel/
```

# Configure Firewall and enable compute nodes to route traffic to the Internet

```
sh ~/week3/firewall.sh
```
