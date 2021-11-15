Connect to the CS74 cluster
```
$ ssh user@cs74.stanford.edu
```

Start an interactive Slurm session
```
$ srun --pty bash
```

Load the Matlab Module
```
$ ml apps/matlab
```

Start Matlab
```
$ matlab
```

Invoke the Matlab Checkpoint program
```
$ test_checkpoint
```

Kill the program at a point greater than 7 iterations and less than 50 using CTRL+C. You should see output similar to this
```
Completed iteration 13
Checkpointing frequency is every  7 iterations.Data updated at iteration  14
Completed iteration 14
Completed iteration 15
Completed iteration 16
Completed iteration 17
Completed iteration 18
Operation terminated by user during test_checkpoint (line 49)
```

Continue the program using the Restart file
```
$ test_restart
```

The output should continue from the last checkpoint data save location continuing to interation 50. Press ```Enter``` to continue, type ```quit``` to exit Matlab.

You can view the README in the same directory where your Matlab files are located (~week6/matlab/checkpoint/) or here on github in the week 7 lab directory. 
