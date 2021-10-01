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

2. Check the dependences of ```lapack``` before you install it:

```
rpm -qpR lapack-3.4.2-8.el7.i686.rpm
```

3. Install the package without using the ```y``` flag:

```
yum install example2.rpm
```

4. Remove the package and any package that dependends on the package:
```
yum remove example2.rpm
```

5. Install the package using the ```y``` flag:
```
yum -y install example2.rpm
```
