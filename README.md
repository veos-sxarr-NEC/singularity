# Singularity for SX-Aurora TSUBASA

This repository has Singularity recipes to build Singularity image to execute a program on Vector Engine of SX-Aurora TSUBASA.

This document explains how to build a Singularity image with VEOS and related software, and how to execute a VE application on Singuraliry using the image.
This document also explains how to build a Singuraliry image with NEC MPI and related software, and how to execute a MPI application on Singuraliry using the image.

We have tested the Singularity recipes with the following version of Singularity.

* Singularity 3.6.4-1.el8

## Build the singularity image of VEOS

Clone the repository.

~~~
$ git clone https://github.com/veos-sxarr-NEC/singularity.git
~~~

Change the current directory to the directory which has Singularity recipes.
~~~
$ cd singularity/CentOS8.2
~~~

Download TSUBASA-soft-release-2.3-1.noarch.rpm.

~~~
$ curl -O https://www.hpc.nec/repos/TSUBASA-soft-release-2.3-1.noarch.rpm
~~~
If your network is behind a proxy, please update dnf.conf to set the proxy.

Build a singularity image.

~~~
$ singularity build --fakeroot veos.sif Singularity
~~~

## Run an application in the singularity container

Run an application using the below command.

~~~
$ singularity exec --bind /var/opt/nec/ve/veos <image SIF> <pass to binary in container>
~~~

For example, run an application with VE NODE#0.
~~~
$ singularity exec --bind /var/opt/nec/ve/veos veos.sif env VE_NODE_NUMBER=0 ./a.out
~~~

## Build the singularity image of NEC MPI

Clone the repository.

~~~
$ git clone https://github.com/veos-sxarr-NEC/singularity.git
~~~

Change the current directory to the directory which has Singularity recipes.
~~~
$ cd singularity/CentOS8.2
~~~

Download TSUBASA-soft-release-2.3-1.noarch.rpm.

~~~
$ curl -O https://www.hpc.nec/repos/TSUBASA-soft-release-2.3-1.noarch.rpm
~~~

Download MLNX_OFED_LINUX.
Following MLNX_OFED_LINUX archive file is needed.
Archive file should remain compressed.

   - RHEL8.2 MLNX_OFED_LINUX-4.9-0.1.7.0-rhel8.2-x86_64.tar (select MLNX_OFED_LINUX-4.9-0.1.7.0-rhel8.2-x86_64.tgz)

If your network is behind a proxy, please update dnf.conf to set the proxy.

Build a singularity image.

~~~
$ singularity build --fakeroot necmpi.sif Singularity.mpi
~~~

## Run a NEC MPI application in the singularity container

Run a NEC MPI application using the below commands.

~~~
$ source /opt/nec/ve/mpi/<nec mpi version in the image>/bin64/necmpivars.sh
$ mpirun <nec mpi options> singularity exec --bind /var/opt/nec/ve/veos <image SIF> <pass to binary in container>
~~~

For example, run a NEC MPI application on interactive shell.
~~~
$ source /opt/nec/ve/mpi/2.11.0/bin64/necmpivars.sh
$ mpirun -hosts host1,host2 -np 2 -ve 0 /usr/bin/singularity exec --bind /var/opt/nec/ve/veos ./necmpi.sif ./a.out
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
mpirun -np 16 /usr/bin/singularity exec --bind /var/opt/nec/ve/veos ~/necmpi.sif ~/a.out

$ qsub job.sh
~~~
