# SGE Job Submission Guide (`qsub`)

On an HPC cluster running Sun Grid Engine (SGE), production calculations should not run on shared login nodes. The **queue scheduler** manages shared computational resources by:

* Accepting job requests from researchers.
* Inspecting available CPU cores, RAM, and hardware features across compute nodes.
* Placing jobs in priority queues and executing them when resources become available.

The primary tool for interacting with the SGE scheduler is `qsub`.

```
                  ┌─────────────────┐
                  │   Login Node    │
                  │ (qsub myscript) │
                  └────────┬────────┘
                           │ Submits job ticket
                           ▼
                  ┌─────────────────┐
                  │  SGE Scheduler  │
                  │ (Match & Queue) │
                  └────────┬────────┘
                           │ Dispatches when slots/RAM free
         ┌─────────────────┼─────────────────┐
         ▼                 ▼                 ▼
  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
  │Compute Node 1│  │Compute Node 2│  │Compute Node 3│
  └──────────────┘  └──────────────┘  └──────────────┘
```

---

## Job Script

While individual binaries can be submitted directly via `qsub -b y /path/to/binary`, creating a **job script** is standard practice. A job script is a shell script that combines scheduler directives with computational execution steps.

### SGE Directive Syntax: `#$`

Lines starting with `#$` are scheduler directives:

* The Linux shell treats `#$` as a standard comment, skipping it during execution.
* The `qsub` utility reads these directives to configure resources when submitting the script.
* **Precedence Rule:** Command-line flags override in-script `#$` directives (e.g., passing `qsub -q UI myscript.job` overrides `#$ -q all.q` inside the file).

### Annotated Job Script Template

```bash
#!/bin/bash
# -------------------------------------------------------------
# SGE SCHEDULER DIRECTIVES
# -------------------------------------------------------------
# Set a descriptive job name (visible in qstat)
#$ -N alignment_run1

# Execute from the directory where qsub was executed
#$ -cwd

# Define the shell interpreter
#$ -S /bin/bash

# Target Queue (e.g., UI, UI-HM, all.q)
#$ -q UI

# Walltime limit (HH:MM:SS) - Terminates job if exceeded
#$ -l h_rt=04:00:00

# Per-slot virtual memory limit (Strict kernel limit)
#$ -l h_vmem=4G

# Parallel Environment: Request 8 shared-memory cores
#$ -pe smp 8

# Output and error logging paths
#$ -o logs/run1_$JOB_ID.out
#$ -e logs/run1_$JOB_ID.err

# Email alerts: begin (b), end (e), and abort (a)
#$ -m bea
#$ -M user@uiowa.edu

# -------------------------------------------------------------
# WORKFLOW COMMANDS
# -------------------------------------------------------------

echo "Job $JOB_ID started on $(hostname) at $(date)"

# Load required application modules
module load biology/samtools/1.18

# Align application threads with allocated slots
export OMP_NUM_THREADS=$NSLOTS

# Run computation
./bin/process_alignments --threads $NSLOTS --input data/sample.bam

echo "Job finished at $(date)"
```

---

## Common `qsub` Submission Options

| Option Flag | Argument | Description & Behavior |
| :--- | :--- | :--- |
| `-cwd` | *None* | **Essential:** Runs the job in the current working directory. Without this, jobs default to your home directory (`$HOME`). |
| `-V` | *None* | Exports all active environment variables from your current login shell to the execution node. |
| `-N` | `<name>` | Assigns an arbitrary name to the job for easier tracking in `qstat`. |
| `-q` | `<queue>` | Specifies target queue: `UI` (default), `UI-HM` (high memory), or `all.q` (opportunistic/preemptible). |
| `-pe` | `<env> <n>` | Parallel environment and slot count (e.g., `smp 8` for multi-threaded jobs on a single host). |
| `-l h_rt=` | `hh:mm:ss` | **Hard Runtime Limit:** Maximum wall clock execution time. Jobs running longer than this are sent `SIGKILL`. |
| `-l h_vmem=` | `<size>` (e.g., `4G`) | **Per-slot virtual memory cap:** Enforced by the kernel. Exceeding this triggers termination. |
| `-l mf=` | `<size>` (e.g., `16G`) | **Free memory gate:** Node matchmaking requirement; ensures a host has free RAM, but does not cap usage. |
| `-o` / `-e` | `<path>` | Custom destination for standard output (`stdout`) and standard error (`stderr`). |
| `-j` | `y` or `n` | Merges `stderr` into the `stdout` file when set to `y`. |
| `-m` | `b,e,a,n` | Notifications: **b**egin, **e**nd, **a**bort, **n**one. (`s` for suspend is unsupported on most clusters). |
| `-M` | `<email>` | Destination email address for notifications. |

---

!!! bug "Key Gotchas & Best Practices"
    Windows Line Endings (`CRLF` vs. `LF`)
    Scripts created or edited on Windows systems use carriage return line endings (`\r\n`). When submitted on Linux, this corrupts the interpreter path:
    ```text
    /bin/sh^M: bad interpreter: No such file or directory
    ```
    * **Fix:** Convert line endings before submitting:
    ```bash
    dos2unix myscript.job
    ```

!!! note "Memory Limits: `h_vmem` vs. `mf` (mem_free)"
    * `mf` / `mem_free` is evaluated **only at scheduling time** to confirm node availability.

### Shell Redirection Precedence
Explicit shell redirection in your script:
```bash
./my_program > local_output.txt
```
takes precedence over `#$ -o`. The `-o` output log will only capture commands executed without explicit file redirection.

---

## Exit Codes & Troubleshooting

When a job terminates, its exit status indicates whether it finished cleanly or was killed. For signal-based terminations, SGE computes the status as:

$$\text{Exit Status} = 128 + \text{Signal Number}$$

| Exit Code | Signal | Common Causes |
| :--- | :--- | :--- |
| `0` | None | Job completed successfully. |
| `1`–`127` | Application Error | Code failure, command not found, or syntax error in your script. |
| **`137`** | `128 + 9 (SIGKILL)` | **1. Out-of-Memory (OOM):** Job exceeded `-l h_vmem`.<br>**2. Forced Eviction:** SGE sent a hard kill signal after preemption. |
| **`143`** | `128 + 15 (SIGTERM)` | **1. Preemption:** Job in `all.q` was evicted by a higher-priority job.<br>**2. Exceeded Walltime:** The job ran past its `-l h_rt` limit.<br>**3. Memory Limit:** Initial grace period termination for memory cap. |

!!! note "Inspecting Historical Job Data via `qacct`"

    To check why a job failed after it leaves the active queue:

    ```bash
    qacct -j <JOB_ID>
    ```

    Inspect these fields in the output:
    
    * `failed`: Displays internal SGE scheduling or eviction codes.
    * `exit_status`: Displays the final numeric code (e.g., `137` or `143`).
    * `maxvmem`: The maximum memory footprint observed during execution. If `maxvmem` is near your requested `h_vmem`, increase the memory allocation.