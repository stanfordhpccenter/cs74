For this week's assignment, we will be installing a number of tools that will add more functionality to our clusters. To complete the assignment, you will need to complete the steps from week 1 (installing CentOS) and week 2 (installing OpenHPC components). After you have those finished, you may begin to follow the instructions on this page.

If you feel that you do not understand the relevance of the tools installed and used in this guide, please refer to previous lectures (which covered tools such as Mellanox OFED and Infiniband in general). Otherwise, do not heasitate to contact course staff through Canvas if you need any clarification.

## First Step

SSH into your cluster:

```
ssh root@hpcc-cluster-[C]
```
## GNU Compilers

Install from `yum`:

```
yum -y install gnu8-compilers-ohpc
```

Modify configuration file to ensure it works correctly on the cluster:

```
perl -pi -e "s/family \"compiler\"//" /opt/ohpc/pub/modulefiles/gnu8/8.3.0
```

## Intel Compilers

Install Intel Parallel Studio dependency:

```
yum -y install libXScrnSaver
```

Copy installation files from `hpcc-cluster` to your cluster (you will be prompted for your password):

```
cd ~
scp -r [user]@hpcc-cluster:/opt/ohpc/pub/cs74/week3 .
```

Change into the installation files directory:

```
cd ~/week3/intel-psxe
```

Uncompress files:

```
tar -zxvf p*.tgz && cd p*
```

Install Intel Parallel Studio XE (note: you will not receive any output for several minutes [10+] after running this command):

```
./install.sh -s ../silent.cfg
```

Note that you may receive some errors/warnings as you run the command. These are generally harmless, but if you find that you are receiving a lot, there may have been a mistake in a previous command.

Install a program that ensures users of a system will be able to access the Intel compilers using modules:

```
yum -y install intel-compilers-devel-ohpc intel-mpi-devel-ohpc
```

Download license from license server:

```
echo 'setenv INTEL_LICENSE_FILE 28518@hpccenter-license.stanford.edu' >> /opt/ohpc/pub/modulefiles/intel/2020.4.912
```
