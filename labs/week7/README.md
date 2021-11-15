```
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% On a cluster such as SCC, jobs that take hours or days to run to completion    %
% on occasion could suffer from system abort in mid-stream due to a variety of   %
% reasons: power failure, walltime limit, scheduled shutdown, to name a few.     %
% Rerunning failed job means inevitable delay and waste of system resources.     %
% Checkpoint Restarting has long been a common technique with which researchers  %
% turn to tackle this issue. In brief, Checkpoint Restarting essentially means   %
% saving data to disk periodically so that if need be, you can restart the job   %
% where data were last saved. Checkpoint Restarting can either be dealt with     %
% through a batch scheduler (if supported) or do it the old-fashioned way by     %
% writing to disk manually yourself. On the SCC, the OGE batch scheduler         %
% checkpointing feature is not supported. A rather simple MATLAB checkpointing   %
% tool has been developed by SCV to approach manual checkpointing systematically.%
% facilitate rerunning of your job should a system abort occurs.                 %
% This MATLAB example code demonstrates the usage of this checkpointing tool to  %
% facilitate rerunning of your job should a system abort occurs.                 %
% The problem is to compute the arithmetic sum                                   %
% A = 1 + 2 + 3 + . . . + N = N*(N+1)/2                                          %
%                                                                                %
% December 8, 2013                                                               %
% Kadin Tseng, SCV, Boston University  (kadin@bu.edu)                            %
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
```
Check list for checkpointing:

1) Before the start of the iteration loop, "check-in" each and every variable
   that should be checkpointed in the event of restarting the job

```
matfile = 'mydata.mat';     % mandatory; name of checkpoint mat-file; your name choice
s = struct();               % mandatory; create struct for checkpointing variables
s = chkin(s,{'s'});         % mandatory; this struct keep tracks of what need saving
s = chkin(s,{'iter'});      % mandatory; iter is the name of the iteration loop index
s = chkin(s,{'frequency'}); % mandatory; frequency is the period of checkpointing, i.e.,
                            % how often you want to perform a save. A finer grain saving
                            % prevents less lost iteration but costs more because of it
s = chkin(s,{'Niter'});     % mandatory; total number of iterations
s = chkin(s,{'a' 'n' 'A'}); % a, n, A are names of variables needed saving

% continue until all variables are checked in. Note that you are only
% checking in the variables, they don't need to have been already defined

chkNames = fieldnames(s);    % the full list of variables to checkpoint
nNames = length(chkNames);   % number of variables in list
```

2) Before the end of the iteration loop, checkpoints once every **frequency** steps

```
if mod(iter, frequency) == 0
   iter    % prints iteration number
   chkpt   % performs checkpointing (save) every **frequency** iterations
end
```

3) Make a copy of your original m-file and modify it to handle checkpoint restarting
   It should contain this . . .

```
   matfile = 'chkpt.mat';          % the mat-file name with which data have been checked
   load(matfile);                  % load data saved previously with chkpt.m
   chkiter = s.iter;               % last saved iteration count
   chkNiter = s.Niter;             % total number of iterations
   for iter=chkiter+1, chkNiter    % start from the next step
      % computations in original program . . .
   end
```

4) Notes:

```
A. For this version, the chkpt.m file is written as a script, rather than function,
   m-file. This makes it simple to implement as everything needed to be checkpointed
   are readily available as it shares the same workspace with the main program.
   Users are advised to exercise care to ensure that variables to be saved have the
   intended values at the time it is being checkpointed. Most importantly, the struct
   array name, declared s in the attached example, is not used for other purposes.

 B. Especially if you have a lot of variables to check-in, you may find it
   appealing to incorporate the whole check-in commands into a SCRIPT m-file to avoid
   clutter.

C. While this checkpointing tool is designed for restarting, you may find it useful
   for other purposes; for instance for the usual output handling.
```
