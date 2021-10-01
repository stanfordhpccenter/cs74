# Package Managers, File Systems, RPMs

Remember to replace ```[C]``` with your assigned cluster number. For example, if you are assigned cluster 10, then ```hpcc-cluster-[C]``` would be entered as ```hpcc-cluster-10```.

SSH into your cluster:

```
ssh hpcc-cluster-[C].stanford.edu
```

### Complete the following exercises on YUM and RPMs

1. Check what's in the ```lapack``` package:
```
rpm  -ql lapack-3.4.2-8.el7.i686.rpm
```

2. Check the dependencies of ```lapack``` before installing it:
```
rpm -qpR lapack-3.4.2-8.el7.i686.rpm
```

3. Install ```lapack``` without using the ```y``` flag:
```
yum install lapack-3.4.2-8.el7.i686.rpm
```

4. Remove ```lapack``` and any packages that depend on lapack:
```
yum remove lapack-3.4.2-8.el7.i686.rpm
```

5. Install ```lapack``` using the ```y``` flag:
```
yum -y install lapack-3.4.2-8.el7.i686.rpm
```

### Install the compute node OS using a VNFS image


