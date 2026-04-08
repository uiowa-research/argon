# Using Apptainer containers on Argon

Containerization of workloads has become increasingly popular, particularly using **Docker**. However, Docker is generally not suitable for HPC applications because it requires root-level access to run. **Apptainer** (formerly Singularity) is the primary alternative for HPC environments, as it allows users to run containers without needing administrative privileges.

Apptainer is fully deployed on the **Argon cluster** and is capable of importing existing Docker containers. For comprehensive technical details, please refer to the [Official Apptainer User Guide](https://apptainer.org/docs/user/latest/). This page covers the basic tenets and provides instructions on how to run a functioning Apptainer container on the Argon system.

## 1\. The Container Workflow

One of the primary uses of containers is to build a specific software stack or a reproducible workflow. With Apptainer, the standard process is:

1.  **Create** an empty container or definition file.
2.  **Bootstrap** a minimal OS into the container.
3.  **Install** packages, dependencies, and import data.
4.  **Copy** the container image to the target run system (Argon).
5.  **Execute** commands in the container via the Apptainer interface.

!!! warning "Where to generate images"

	Steps 1–3 require **root** (administrator) access. This means you must perform these steps on a Linux machine that you manage. You **cannot build** containers on the Argon cluster; you can only **run** them. All modifications must happen on your local Linux machine. (Windows or Mac users can use VirtualBox or WSL2 to set up a local Linux environment for building).

## 2\. File Systems and Home Directories

Your home and scratch directories (`/nfsscratch` and `localscratch`) are accessible from within the container when run on Argon.

  * **Potential Conflict:** Using the host's home directory can sometimes present issues, as the "dot files" (e.g., `.bashrc`) in your Argon home directory may not be compatible with the container environment.
  * **The Solution:** Use the `-H` flag to specify an alternate home directory inside the container. This limits the scope of your home directory to a clean space. Plan your data paths accordingly.

## 3\. GPU Usage and Kernel Compatibility

If you are running a GPU-accelerated job, the code inside the container must interface with the **NVIDIA kernel module** on the host system.

  * **Portability Variable:** If you run the same container on different host systems, the kernel module versions may differ. Even if the version is the same, it may have been built against a different OS kernel.
  * **Recommendation:** Keep this in mind if your primary goal is perfect reproducibility, as the host's NVIDIA kernel module remains a variable in your environment.

## 4\. Example: Ubuntu Container Definition

The following is a basic example of a Ubuntu container definition file. This header represents the minimum required to create the container OS.

```bash
BootStrap: debootstrap
OSVersion: bionic
MirrorURL: http://us.archive.ubuntu.com/ubuntu/
```

## 5\. Running Jobs via the SGE Scheduler

Regardless of whether a container job is MPI-based or serial, it should be submitted to the **SGE queue system** like any other resource-intensive job. Apptainer is scheduler-agnostic, meaning it will run in SGE without configuration changes—you simply run your commands through the Apptainer interface.

### Using %runscript

You can specify a `%runscript` in your definition file. If your container is dedicated to a single process, you can execute the container directly to trigger the workflow. The runscript can also accept parameters:

```bash
# Execute a container with a defined runscript
apptainer run my_container.sif [parameters]
```

## 6\. Support Policy

We provide Apptainer on Argon to facilitate containerized workflows and custom software stacks. While Research Services strives to ensure the Apptainer runtime works smoothly on the cluster, **building and maintaining the internal contents of the containers is the responsibility of the end user.**

