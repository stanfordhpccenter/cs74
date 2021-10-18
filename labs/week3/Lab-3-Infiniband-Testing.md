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
