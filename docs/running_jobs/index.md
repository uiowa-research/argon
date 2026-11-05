# Running Jobs

## System Architecture: Shared Resources & Access Models

High-Performance Computing (HPC) clusters are shared supercomputing environments aggregating thousands of compute cores, multi-terabyte memory pools, and GPU accelerators across hundreds of networked nodes.

Because researchers share these physical assets concurrently, resources cannot be assigned on an unmanaged basis. Access is mediated by **Sun Grid Engine (SGE) Workload Manager**, which acts as cluster coordinator, scheduler, and policy enforcer.

```
                      ┌─────────────────────────────────────────┐
                      │              User Access                │
                      │  • Terminal / SSH                       │
                      └────────────────────┬────────────────────┘
                                           │
                                           ▼
                      ┌─────────────────────────────────────────┐
                      │               Login Nodes               │
                      │  • Text editing, git, basic file moves  │
                      │  • Submitting jobs via Scheduler        │
                      │  • STRICTLY NO COMPUTATION              │
                      └────────────────────┬────────────────────┘
                                           │
                             Dispatched by SGE Scheduler
                                           │
         ┌─────────────────────────────────┼─────────────────────────────────┐
         ▼                                 ▼                                 ▼
┌──────────────────┐              ┌──────────────────┐              ┌──────────────────┐
│  Compute Node 1  │              │  Compute Node 2  │              │  Compute Node N  │
│  (Batch Job)     │              │  (Interactive)   │              │  (Job Array)     │
└──────────────────┘              └──────────────────┘              └──────────────────┘
```

!!! danger "DO NOT run jobs on Login Nodes"
    A login node is the entry point where all users land and carry out lightweight tasks, such as editing code, debugging, and navigating files. Heavy computation on a login node slows the system down for everyone.
    
    
---

## The Login Node Policy: Boundaries & Enforcement

### The Role of Login Nodes
Login nodes are the shared entry point of the cluster. Every SSH user lands on one of a small pool of login hosts. Their capacity is shared by dozens or hundreds of concurrent users at any given moment.

| Allowed on Login Nodes | Prohibited on Login Nodes |
| :--- | :--- |
| Editing code and job scripts (`vim`, `nano`, VS Code) | Running Python, R, MATLAB, or C/C++ simulations |
| Managing files (`ls`, `cp`, `mv`, `tar`, `rsync`) | Compiling large, complex multi-threaded software stacks |
| Checking queue and job states (`qstat`, `qacct`) | Executing data preprocessing or parsing on large datasets |
| Submitting jobs (`qsub`,`qrsh`,`qlogin`) | Running unthrottled submission loops |
