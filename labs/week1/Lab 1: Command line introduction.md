# Command Line Intro Exercises

### Access the remote server

1. Open your terminal. 
- If you're running MacOS or a Linux distribution OS, then you should already have a Terminal app.
- If you're running a Windows OS, find Windows PowerShell or Windows Terminal by searching for either in the Start box.

2. In the command line, SSH into the remote server using the username and password that we provide. Replace ```[user]``` with your assigned username.

```
ssh [user]@hpcc-cluster.stanford.edu
```

You'll be prompted for your password. Type in your password and press enter. Your keystrokes will not be visible on the command line.

You might get a warning message asking whether you trust the 

### Complete the following exercises

1. Create a directory in your home directory named ```slurm-jobs```.

```
mkdir slurm-jobs
```

2. Create a file in ```slurm-jobs``` named ```myscript```

```
nano myscript
```

3. In the file, enter the following and save changes to the file:

```
#!/bin/sh
#SBATCH --time=1
echo $HOME
```

4. Add the following line ```#SBATCH --mail-su=user@domain.edu``` to the end of ```myscript``` without opening the file:

```
echo "#SBATCH --mail-su=user@domain.edu"
```
