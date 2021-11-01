```
yum -y install charliecloud-ohpc singularity-ohpc
```


```
wwsh -y provision set compute-* --kargs="${kargs} namespace.unpriv_enable=1"
```


```
echo "user.max_user_namespaces=15076" >> $CHROOT/etc/sysctl.conf
```


```
wwsh file import /etc/subuid
wwsh file import /etc/subgid
```


```
wwsh -y provision set "compute-*" --vnfs=centos7 --bootstrap=`uname -r` --files=dynamic_hosts,passwd,group,shadow,network,slurm.conf,munge.key,nhc.conf,subuid,subgid
```


```
wwvnfs --chroot=$CHROOT
```
