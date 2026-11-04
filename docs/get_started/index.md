# Getting Started with HPC 

<link rel="stylesheet" href="../assets/stylesheets/buttons.css">

## What is an HPC?

A High Performance Computing (HPC) cluster is a group of uniformly configured compute nodes that work together to solve large-scale computational problems. The nodes are typically connected by high-bandwidth, low-latency networks, 
so they function as a single unified system. This lets researchers tackle problems that need more memory or processing power than a typical workstation or single server can provide.

An HPC cluster typically includes:

- **Compute nodes:** Anywhere from a few to thousands, these perform the actual computations.
- **Login or staging nodes:** Used to access the system and prepare jobs.
- **Head nodes:** Manage job scheduling, resource allocation, and data coordination.

---

## What is a Scheduler?

A scheduler is a <mark>software tool that manages cluster resources and distributes them fairly among users.</mark> It works together with a queuing system, which organizes jobs based on available resources or user access.

When submitting a job, you can use the scheduler to request:

- **Parallel environments (PEs):** For jobs that require communication between nodes.
- **Special resources:** Such as GPUs, co-processors, or extra memory.

The scheduler monitors job submissions and decides which jobs can run based on current resource availability. It may also prioritize jobs from users who use the system less often, 
which ensures equitable access and prevents any one user from monopolizing resources.

---

## Roadmap for utilizing HPC cluster from scratch

1. Learn Basic Linux
    Our cluster runs CentOS Linux. To use it effectively, you'll need basic command-line skills—like navigating directories and editing files.

    - Recommended resource: The Linux Command Line ([free PDF](https://linuxcommand.org/tlcl.php))
    - Quick reference: [Linux Cheat Sheet](https://uiowa.atlassian.net/wiki/download/attachments/76513414/the-linux-command-line.pdf?version=1&modificationDate=1412807008197&cacheVersion=1&api=v2)

2. Map Your Work to the Cluster

    If your project:

    - Requires more memory than a desktop,
    - Needs fast turnaround,
    - Benefits from scheduled execution,

    ...then HPC may be a good fit.

    Ask yourself:

    - Does your code run on Linux?
    - Can it be executed in batch mode?
    - Is it a parallel (HPC) or serial (HTC) job?

3. Consider Compilation Needs

    If you're migrating code from another system, you may need to recompile it—especially if it uses MPI.
    See our notes on [compiling software](https://uiowa.atlassian.net/wiki/spaces/hpcdocs/pages/76513446/Compiling+Software)

4. Check Software Availability

    Browse our [Installed Software List](https://uiowa.atlassian.net/wiki/spaces/hpcdocs/pages/76513438/Software+Installations). If your required package isn’t listed, contact us. We may install it centrally or assist with installing it in your home directory.

5. Estimate Resource Requirements

    Knowing how much memory or how many processes your job needs helps ensure successful execution. Try running a small version of your job first, then scale up. You can also use our **development queue** for testing.






<!--
<html>
<a href="/get_started/..next..page"><button class="right-button" style="float: right;"></button></a>
</html>

-->
<br>
