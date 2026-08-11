# Transition from SGE to SLURM


User Commands | SGE | SLURM
-------------- | --- | -----
Interactive login | qlogin | srun --pty bash 
Job submission | qsub [script_file] | sbatch [script_file]
Job deletion | qdel [job_id] | scancel [job_id]
Job status by job | qstat -j [job_id] | squeue [job_id]
Job status by user | qstat -u [user_name] | squeue -u [user_name]
