# Command Line Intro Exercises

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
