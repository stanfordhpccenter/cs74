**NOTE: All commands below are performed as a regular user on your cluster**

## Install GROMACS

Load modules
```
module purge
ml pmix cmake gnu8 intel/19.1.3.304 impi
```

Optional: check modules
```
ml
```

Optional: check MPI-C and MPI-CXX compiler binding
```
mpicc -show
mpicxx -show
```

Obtain GROMACS src
```
scp -r <user>@hpcc-cluster:/opt/ohpc/pub/cs74/week4 ~/
```

Build and install GROMACS
```
cd week4/gromacs
tar -zxvf gromacs-2020.2.tar.gz && cd gromacs-2020.2
mkdir build install run && cd build

cmake .. -DGMX_FFT_LIBRARY=mkl -DMKL_LIBRARIES=-mkl \
        -DMKL_INCLUDE_DIR=$MKLROOT/include \
        -DGMX_SIMD=AVX_256 \
        -DGMX_MPI=ON \
        -DGMX_BUILD_MDRUN_ONLY=on \
        -DBUILD_SHARED_LIBS=off \
        -DGMX_HWLOC=off \
        -DCMAKE_INSTALL_PREFIX=../install \
        -DCMAKE_C_COMPILER=mpicc -DCMAKE_CXX_COMPILER=mpicxx

make -j 4
make install
```

Optional check installation
```
ls ../install
```

Add the bin directory to PATH
```
echo 'export gmx_root_dir=$HOME/week4/gromacs/gromacs-2020.2' >> ~/.bashrc
echo 'PATH=$PATH:$gmx_root_dir/install/bin' >> ~/.bashrc
source ~/.bashrc
```

Optional: quick check
```
which mdrun_mpi
```

###### GROMACS test run ######
Copy stmv.tpr to run dir
```
cp ~/week4/gromacs/stmv.tpr $gmx_root_dir/run
```

Run GROMACS
```
cd $gmx_root_dir/run
```

Run with single process (note the code is running with OpenMP)
```
mdrun_mpi -v -s stmv.tpr -nsteps 2 -noconfout -nb cpu -pin on
```

Run with multiple MPI-processes
```
mpiexec mdrun_mpi -v -s stmv.tpr -nsteps 10 -noconfout -nb cpu -pin on
```

###### Submit a simulation using a slurm script ######
```
#!/bin/bash
#SBATCH --job-name="GMX"                    # Job name
#SBATCH --nodes=1                           # Number of nodes requested
#SBATCH --ntasks=16                         # Number of processes
#SBATCH --time=00:20:00                     # Time limit request

date
module purge
ml pmix cmake gnu8 intel/19.1.3.304 impi
EXEC_DIR=$gmx_root_dir/run

mpiexec mdrun_mpi -v -s $EXEC_DIR/stmv.tpr -nsteps 100 -c confout.gro -o -nb cpu -pin on

date
```

###### Result visualization ######

Install NGLView and Jupyter using PIP (only need for once)
```
pip --version # optional: check PIP version
```

Add bin directory PATH for Intel Python
```
echo 'PATH=/opt/ohpc/pub/compiler/intel/intelpython3/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Re-check PIP
```
python -m pip --version # optional: check PIP version
```

Install Jupyter and dependencies:
```
python -m pip install jupyter --user
python -m pip install pytraj --user
python -m pip install nglview --user
python -m pip list # optional: check package installation
jupyter nbextension enable --py widgetsnbextension
jupyter nbextension enable nglview --py
```


##### Slurm script: jupyter_submit.slurm #####

```
#!/bin/bash
#SBATCH --job-name="Jupyter"                # Job name
#SBATCH --partition=normal                  # Node partition
#SBATCH --nodes=1                           # Number of nodes requested
#SBATCH --ntasks=1                          # Number of processes
#SBATCH --time=01:00:00                     # Time limit request

date
module purge
ml pmix cmake gnu8 intel/19.1.3.304 impi
EXEC_DIR=$gmx_root_dir/run
hostname && jupyter-notebook --no-browser --notebook-dir=$EXEC_DIR
```

Launch Jupyter notebook
```
sbatch jupyter_submit.slurm
```

Check job
```
squeue -u [username] # note the job ID
```

Gather the data needed to use in following commands. One way to do this is the following:
```
egrep -w 'compute|localhost'  slurm-*.out
```

Output may look like this:
```
compute-1-1
[I 16:44:21.601 NotebookApp]   http://localhost:8888/?token=8ce4b320473a1a9fb3536368c2e5f547c966bf7c3f9861  52
```

##### Access Jupyter from local device #####

The following is an example of port redirection to interact with the jupyter notebook on your local computer:
```
ssh -L <port>:localhost:<port> [user]@cs74.stanford.edu ssh -L <port>:localhost:<port> [user]@hpcc-cluster-[N] -t ssh -N -L <port>:localhost:<port> <compute-node>
```

For example connecting from your computer to hpcc-cluster with a Slurm job executing on compute-1-1 using port 8888 for the Jupyter notebook session:
```
ssh -L 8888:localhost:8888 [user]@cs74.stanford.edu  ssh -L 8888:localhost:8888 [user]@hpcc-cluster-[N] -t ssh -N -L 8888:localhost:8888 compute-1-1
```

Once you have an active port redirection using the former step, open a browser and paste the URL:
```
“http://localhost:<port>/?token=<token>”
```

For example using the output from slurm.*.out above this is the URL:
```
http://localhost:8888/?token=8ce4b320473a1a9fb3536368c2e5f547c966bf7c3f9861
```

Create a new notebook from "New"->"Python3"
Rename the title

Visualize molecular structure
Type the following Python script and run

```
import nglview as nv
view = nv.show_structure_file("confout.gro")
view.clear()
view.add_cartoon()
view
```
##### Result visualization on cs74.stanford.edu #####

Connect to cs74
```
ssh [user]@cs74.stanford.edu
```

Copy files to your home directory
```
cp -r /opt/ohpc/pub/cs74/week4 .
```

Change directory
```
cd ~/week4/jupyter
```

Submit job to the queue
```
sbatch jupyter_submit.slurm
```

Gather the data needed to use in following commands. One way to do this is the following:
```
egrep -w 'compute|localhost'  slurm-*.out
```

Note: You may need to wait a few minutes for data to generate from command above

Output may look like this:
```
compute-1-1
[I 16:44:21.601 NotebookApp]   http://localhost:8888/?token=8ce4b320473a1a9fb3536368c2e5f547c966bf7c3f9861  52
```

##### Access Jupyter on cs74 from local device #####

The following is an example of port redirection to interact with the jupyter notebook on your local computer:
```
ssh -L <port>:localhost:<port> [user]@cs74.stanford.edu -t ssh -N -L <port>:localhost:<port> <compute-node>
```

For example connecting from your computer to cs74 with a Slurm job executing on compute-5-1 using port 8888 for the Jupyter notebook session:
```
ssh -L 8888:localhost:8888 [user]@cs74.stanford.edu -t ssh -N -L 8888:localhost:8888 compute-5-1
```

Once you have an active port redirection using the former step, open a browser and paste the URL:
```
“http://localhost:<port>/?token=<token>”
```

For example using the output from slurm.*.out above this is the URL:
```
http://localhost:8888/?token=8ce4b320473a1a9fb3536368c2e5f547c966bf7c3f9861
```

Create a new notebook from "New"->"Python3"
Rename the title

Visualize molecular structure
Type the following Python script and ```run```

```
import nglview as nv
view = nv.show_structure_file("confout.gro")
view.clear()
view.add_cartoon()
view
```
