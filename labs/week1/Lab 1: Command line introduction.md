# Command Line Introduction

### Access the remote server

1. Open your terminal. 
- If you're running MacOS or a Linux distribution OS, then you should already have a Terminal app.
- If you're running a Windows OS, find Windows PowerShell or Windows Terminal by searching for either in the Start box.

2. In the command line, SSH into the remote server using the username and password that we provide. Replace ```[user]``` with your assigned username.

```
ssh [user]@hpcc-cluster.stanford.edu
```

You'll be prompted for your password. Type in your password and press enter. Your keystrokes will not be visible on the command line.

You might get a warning asking whether you trust the authenticity of the host. Type yes and press enter.

3. Connect to your assigned cluster. Ensure you replace ```[C]``` in the instructions with your assigned cluster number. For example, if your cluster number is 15, you'd enter ```ssh [user]@hpcc-cluster-15.stanford.edu```.

```
ssh [user]@hpcc-cluster-[C].stanford.edu
```

### Complete the following exercises

1. Create a directory in your home directory named ```slurm-jobs```. By default, you'll ssh into your home directory. Your home directory is indicated in the command prompt with the symbol "~".

```
mkdir slurm-jobs
```

2. Create a file in ```slurm-jobs``` named ```myscript```. This will open a new blank file.

```
cd slurm-jobs
nano myscript
```

3. In the file, type in the following, save changes to the file, and then close the file:

```
#!/bin/sh
#SBATCH --time=1
echo $HOME
```

In Windows, save changes to the file and close the file by pressing the "Ctrl" key and "X" key. 
You'll be asked whether to save a modified buffer. Type in yes and press enter.
You'll be asked what file name to write. The current file name should be there by default. Press the enter key.

4. Add the following line ```#SBATCH --mail-su=user@domain.edu``` to the end of ```myscript``` without opening the file:

```
echo "#SBATCH --mail-su=user@domain.edu"
```
