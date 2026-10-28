# Interactive Jobs with `qlogin` (SGE / Argon HPC)

`qlogin` provides an **interactive compute shell** through Sun Grid Engine (SGE). Instead of queueing an asynchronous shell script with `qsub` and waiting for log files to appear, `qlogin` instructs the scheduler to:

1. Allocate requested hardware resources on compute nodes matching your criteria.
2. Establish an active, direct SSH connection to the primary compute node.
3. Provide an interactive shell prompt running on compute hardware identical to production nodes.

### Primary Use Cases

* **Prototyping & Debugging:** Step through scripts, inspect memory issues, and debug execution line-by-line without submitting numerous batch jobs.
* **Hardware-Specific Profiling & Benchmarking:** Test workloads on large-memory nodes (`-l mem=...`), specific CPU microarchitectures, or dedicated queues prior to launching large-scale production runs.
* **Protecting Shared Login Nodes:** Move compilation, heavy pre-processing, large file compression, and CPU-intensive testing off shared login nodes so they remain responsive for all users.

---

## Resource Allocation Mechanics & Policies

| Policy / Constraint | Rule | Reason / Impact |
| :--- | :--- | :--- |
| **Default Allocation** | Attempts to allocate as many idle slots as possible on a node (minimum 1). | Increases available resources for experimentation, but **does not guarantee full-node exclusivity**. |
| **Explicit Requests** | Explicitly define resources (e.g., `-pe smp 4`, `-l h_vmem=16G`). | Prevents unexpected sharing or under-allocation of needed resources. |
| **No Idle Node Reserving** | Do not keep sessions open when not actively working. | Allocated slots count against cluster utilization and block other researchers from running jobs. |
| **Oversubscription Rules** | Do not launch more processes or threads than requested slots. | Nodes may be co-located with other jobs; oversubscribing degrades performance for all users on that node. |
| **Wall Clock Limits** | Default runtime: **24 hours**. Override via `-l h_rt=HH:MM:SS`. | Sessions terminate automatically when reaching the limit. Keep durations realistic to minimize queue impact. |

---

## Graphical Interface Support (<mark>X11 Forwarding</mark>)

You can launch GUI-driven development tools (e.g., MATLAB, RStudio, visualization packages) inside a `qlogin` session:

```
[Local Machine]
       │
       │  ssh -Y argon.hpc.uiowa.edu (or FastX Desktop)
       ▼
[Login Node]
       │
       │  qlogin [resource directives]
       ▼
[Compute Node (X11 DISPLAY forwarded back to client)]
```

* **Via Terminal:** Connect with trusted X11 tunneling using `ssh -Y HawkID@argon.hpc.uiowa.edu`, then launch your `qlogin` session.
* **Via Web/Remote Desktop (FastX):** If connecting through FastX, X11 forwarding is already established inside your session automatically.

!!! note "Note"
    Direct X11 rendering requires high bandwidth and low latency; it is best suited for on-campus connections or low-latency VPN links.

---

## Architectural Difference: Batch Environment vs. `qlogin`

Understanding the difference between batch and interactive environments is critical for managing environment variables and MPI workloads:

```
Batch Job (qsub):
[SGE Daemon] ── spawns job ──► Populates SGE environment ($JOB_ID, $PE_HOSTFILE, etc.)
                               MPI detects scheduler environment natively.

Interactive Job (qlogin):
[SGE Daemon] ── reserves slots ──► Opens standalone SSH session ──► Fresh, unpopulated shell
                                                                  (No SGE runtime variables)
```

Because `qlogin` connects through a standard SSH daemon on the compute node:
* None of the typical `$JOB_ID`, `$SGE_O_WORKDIR`, or queue-specific variables are set in the interactive shell.
* Automatic environment passing (`qsub -V`) does not apply.
* Distributed parallel toolchains like OpenMPI cannot automatically detect the SGE environment or its allocated host topology without manual configuration.

---

## Exiting the Session

To exit an interactive session, run:

```bash
exit
```

This closes the SSH connection, terminates the job on the scheduler, and releases the allocated compute resources immediately for other researchers.

---

## Troubleshooting: Socket Timeouts & Allocation Delays

- Typical Error Message
```text
Your job 3747412 ("QLOGIN") has been submitted
waiting for interactive job to be scheduled ...
timeout (5 s) expired while waiting on socket fd 4
.timeout (10 s) expired while waiting on socket fd 4
.
```
- Causes
    * **Unavailable or Restricted Queue:** Requesting resources in a queue to which you lack permissions (such as an Investor partition).
    * **Resource Exhaustion:** Asking for resources that cannot be satisfied immediately by the scheduler.
    * **Lingering Socket Handshake:** When an initial request fails, the underlying socket remains active for a short grace window. Repeated submission attempts will produce the same timeout message until the socket clears.

- Resolution
    * Double-check your queue name, memory limits, and group memberships.
    * Wait **2 to 5 minutes** for the previous socket connection to fully close before reissuing the `qlogin` command.