# Package Managers, File Systems, RPMs

Remember to replace ```[C]``` with your assigned cluster number. For example, if you are assigned cluster 10, then ```hpcc-cluster-[C]``` would be entered as ```hpcc-cluster-10```.

SSH into your cluster. Remember to first enter through the login node:

```
ssh hpcc-cluster-[C].stanford.edu
```

### Complete the following exercises on YUM and RPMs

1. Install ```singularity``` without using the ```y``` flag:
```
yum install singularity
```

2. Remove ```singularity``` and any packages that depend on lapack:
```
yum remove singularity
```

3. Install ```singularity``` using the ```y``` flag:
```
yum -y install singularity
```

4. Check what's in the ```singularity``` package:
```
rpm -ql singularity
```

5. Check the dependencies of ```singularity```:
```
rpm -qR singularity
```
6. View location of singularity
```
which singularity
```

### Install the compute node OS using a VNFS image

1. Install ```singularity``` using the ```y``` flag:
```
yum -y --installroot=$CHROOT install singularity
```

2. Re-assemble VNFS image
```
wwvnfs --chroot $CHROOT
```

3. Restart the compute node(s):
```
wwsh ssh compute-* reboot
```
4. View location of singularity
```
chroot $CHROOT
which singularity
```

### Install a package in a shared location

1. Install the ```ohpc``` singularity package in a shared NFS location ```/opt/ohpc/pub/~```
```
yum -y install singularity-ohpc
```

2. Load the ```singularity``` environment module:
```
ml singularity
```

3. View location of singularity
```
which singularity
```

4. Purge loaded modules
```
ml purge
```

5. View location of singularity
```
which singularity
```
