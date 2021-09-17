# Package Managers, File Systems, RPMs

Remember to replace ```[C]``` with your assigned cluster number. For example, if you are assigned cluster 10, then ```hpcc-cluster-[C]``` would be entered as ```hpcc-cluster-10```.

SSH into your cluster:

```
ssh hpcc-cluster-[C].stanford.edu
```

### Complete the following exercises

1. Check the dependences of an RPM package before you install it:

```
rpm -qpR example.rpm
```

2. Install the package without using the ```y``` flag:

```
yum install example2.rpm
```

3. Remove the package and any package that dependends on the package:
```
yum remove example2.rpm
```

4. Install the package using the ```y``` flag:
```
yum -y install example2.rpm
```
