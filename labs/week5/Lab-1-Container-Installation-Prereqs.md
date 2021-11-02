First install Charliecloud and Singularity from the OpenHPC repository as this includes module support:
```
yum -y install charliecloud-ohpc singularity-ohpc
```

Enable unprivileged namespaces on the compute node provisioning process:
```
wwsh -y provision set compute-* --kargs="${kargs} namespace.unpriv_enable=1"
```

Set an environment variable controlling number of user namespaces:
```
echo "user.max_user_namespaces=15076" >> $CHROOT/etc/sysctl.conf
```

Import the subuid and subgid files in to the warewulf database. This allows users to create their own "sandboxes" 
```
wwsh file import /etc/subuid
wwsh file import /etc/subgid
```


Append the provision command with the imported files:
```
wwsh -y provision set "compute-*" --vnfs=centos7 --bootstrap=`uname -r` --files=dynamic_hosts,passwd,group,shadow,network,slurm.conf,munge.key,nhc.conf,subuid,subgid
```

Reassemble the VNFS image and import in to the warewulf database:
```
wwvnfs --chroot=$CHROOT
```

Reboot your compute node:
```
ssh compute-1-1 reboot
```
