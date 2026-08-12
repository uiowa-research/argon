# Transition from SGE to SLURM


## Basic Job commands

User Commands | SGE | SLURM
-------------- | --- | -----
Interactive login | qlogin | srun --pty bash 
Job submission | qsub [script_file] | sbatch [script_file]
Job deletion | qdel [job_id] | scancel [job_id]
Job status by job | qstat -j [job_id] | squeue [job_id]
Job status by user | qstat -u [user_name] | squeue -u [user_name]


## Environmental Variables

Variables | SGE | SLURM
--------- | --- | ------
Job ID | $JOB_ID | $SLURM_JOBID
Job Array Index | $SGE_TASK_ID | $SLURM_ARRAY_TASK_ID

## Job Directives

Directives | SGE | SLURM
---------- | --- | -----
Script directive | #$ | #SBATCH
queue | -q [queue] | -p [queue]
Wall clock limit | -l h_rt=[seconds] | -t [min] OR -t [days-hh\:mm\:ss]
Standard out file | -o [file_name] | -o [file_name]
Standard error file | -e [file_name] | -e [file_name]
Combine STDOUT & STDERR files | -j yes | (use -o without -e)
Event notification | -m abe | –mail-type=[events]
send notification email | -M [address] | –mail-user=[address]
Job name | -N [name] | –job-name=[name]
Set working directory | -wd [directory]	| –workdir=[dir_name]
Job arrays | -t [array_spec] | –array=[array_spec] 
