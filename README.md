# Apptainer for SX-Aurora TSUBASA

This repository has Apptainer recipes to build Apptainer image to execute a program on Vector Engine of SX-Aurora TSUBASA.

This document explains how to build a Apptainer image with VEOS and related software, and how to execute a VE application on Apptainer using the image.
This document also explains how to build a Apptainer image with NEC MPI and related software, and how to execute a MPI application on Apptainer using the image.

You can save and use the image as execution environment for your program.

We have tested the Apptainer recipes with the following version of Apptainer.

RHEL 8.10 
* Apptainer 1.3.4-1.el8

RHEL 10.0
* Apptainer 1.4.3-1.el10_0


## Note

The apptainer-suid package must be installed to run the VE program.

If you run the VE program without apptainer-suid, it will fail as follows.

~~~
$ apptainer exec --bind /var/opt/nec/ve/veos veos.sif ./a.out
VE mmap failed on INTERP - TEXT: Bad address
Aborted (core dumped)
~~~

## Compatibility problems

To avoid the compatibility problem, please consider the below points:

* The compatibility of software between a host machine and a container.
  * The version of VEOS in a host machine must be greater than or equal to the version of VEOS in a container.
  * The version of NEC MPI in a host machine must be greater than or equal to the version of NEC MPI in a container.
  * The version of MOFED in a host machine must be equal to the version of MOFED in a container.

* The compatibility of software between a container and a build machine
  * The version of NEC MPI in a container must be greater than or equal to the version of NEC MPI in a build machine where you built your program.
  * Each software version of NEC SDK in a container must be greater than or equal to each software version of NEC SDK in a build machine where you built your program.


## Build the apptainer image of VEOS

Clone the repository.

~~~
$ git clone https://github.com/veos-sxarr-NEC/singularity.git
~~~

There are directories containing recipes and necessary files to create images for specific operating systems. Change the current directory to the directory for the desired operating system. For example: singularity/RockeyLinux10.

~~~
$ cd singularity/RockyLinux10
~~~
Download TSUBASA-soft-release-ve1-3.1-2.noarch.rpm

~~~
$ curl -O https://sxauroratsubasa.sakura.ne.jp/repos/TSUBASA-soft-release-ve1-3.1-2.noarch.rpm
~~~

If your network is behind a proxy, please update dnf.conf to set the proxy.

Build a apptainer image.

~~~
$ apptainer build --fakeroot veos.sif veos.def
~~~

If the build fails because of issues related to resolv.conf, follow the steps below to build.
~~~
$ touch resolv.conf
$ apptainer build --fakeroot -B ${PWD}/resolv.conf:/etc/resolv.conf veos.sif veos.def
~~~

## Run an application in the apptainer container

Run an application using the below command.

~~~
$ apptainer exec --bind /var/opt/nec/ve/veos <image SIF> <pass to binary in container>
~~~

For example, run an application with VE NODE#0.
~~~
$ apptainer exec --bind /var/opt/nec/ve/veos veos.sif env VE_NODE_NUMBER=0 ./a.out
~~~

## Run a VEO or VEDA application in the apptainer container on RHEL10

Non-root user needs the CAP_IPC_LOCK capability to run VEO and VEDA programs, but this is required only on RHEL10.
Please execute the following command before running VEO or VEDA programs. This step must be done once by the root user.

~~~
$ sudo apptainer capability add --user <user> IPC_LOCK
~~~

Then, execute the following command with "--add-caps" option to add the Linux capability.

~~~
$ apptainer exec --bind /var/opt/nec/ve/veos --add-caps CAP_IPC_LOCK <image SIF> <pass to binary in container>
~~~

## Build the apptainer image of NEC MPI

Clone the repository.

~~~
$ git clone https://github.com/veos-sxarr-NEC/singularity.git
~~~

Change the current directory to the directory which has Apptainer recipes.
~~~
$ cd singularity/RockyLinux10
~~~

Download TSUBASA-soft-release-ve1-3.1-2.noarch.rpm.

~~~
$ curl -O https://sxauroratsubasa.sakura.ne.jp/repos/TSUBASA-soft-release-ve1-3.1-2.noarch.rpm
~~~

### For RHEL8.10
Download MLNX_OFED_LINUX.
Following MLNX_OFED_LINUX archive file is needed.
Archive file should remain compressed.

   - MLNX_OFED_LINUX-23.10-3.2.2.0-rhel8.10-x86_64.tgz

If your network is behind a proxy, please update dnf.conf to set the proxy.

Build a apptainer image.

~~~
$ apptainer build --fakeroot necmpi.sif necmpi.def
~~~

If the build fails because of issues related to resolv.conf, follow the steps below to build.
~~~
$ touch resolv.conf
$ apptainer build --fakeroot -B ${PWD}/resolv.conf:/etc/resolv.conf necmpi.sif necmpi.def
~~~

### For RHEL10.0
Download NVIDIA DOCA_LINUX.
Following NVIDIA DOCA_LINUX is needed.

   - doca-host-3.1.0-091000_25.07_rhel100.x86_64.rpm

If your network is behind a proxy, please update dnf.conf to set the proxy.

Build a apptainer image.

~~~
$ apptainer build --fakeroot necmpi.sif necmpi.def
~~~

If the build fails because of issues related to resolv.conf, follow the steps below to build.
~~~
$ touch resolv.conf
$ apptainer build --fakeroot -B ${PWD}/resolv.conf:/etc/resolv.conf necmpi.sif necmpi.def
~~~


## Run a NEC MPI application in the apptainer container

Run a NEC MPI application using the below commands.

~~~
$ source /opt/nec/ve/mpi/<nec mpi version in the image>/bin64/necmpivars.sh
$ mpirun <nec mpi options> /usr/bin/apptainer exec --bind /var/opt/nec/ve/veos <image SIF> <pass to binary in container>
~~~

For example, run a NEC MPI application on interactive shell.
~~~
$ source /opt/nec/ve/mpi/2.11.0/bin64/necmpivars.sh
$ mpirun -hosts host1,host2 -np 2 -ve 0 /usr/bin/apptainer exec --bind /var/opt/nec/ve/veos ./necmpi.sif ./a.out
~~~

For example, run a NEC MPI application with NQSV.
necmpi.sif and a.out is located in your home directory.
~~~
$ vi job.sh
#!/bin/bash
#PBS -q  bq
#PBS -T necmpi
#PBS -b 2
#PBS -l elapstim_req=300
#PBS --venode=2 --cpunum-lhost=2

source /opt/nec/ve/mpi/2.11.0/bin64/necmpivars.sh
mpirun -np 16 /usr/bin/apptainer exec --bind /var/opt/nec/ve/veos ~/necmpi.sif ~/a.out

$ qsub job.sh
~~~

## Run a VEO or VEDA application in the apptainer container on RHEL10

Non-root user needs the CAP_IPC_LOCK capability to run VEO and VEDA programs, but this is required only on RHEL10.
Please execute the following command before running VEO or VEDA programs. This step must be done once by the root user.

~~~
$ sudo apptainer capability add --user <user> IPC_LOCK
~~~

Then, execute the following command with "--add-caps" option to add the Linux capability.

~~~
$ apptainer exec --bind /var/opt/nec/ve/veos --add-caps CAP_IPC_LOCK <image SIF> <pass to binary in container>
~~~
