# nzesm_apptainer
Definition files for creating an UM/NZESM Apptainer environment

## Introduction

Containerisation allows you to compile code that can be easily ported from one platform to another (with some restrictions). As the name suggests, all the dependencies are contained -- there are no undefined references.

The container `umenv_intel2004` described below comes with an operating system, compilers, libraries (MPI, NetCDF) and tools (fcm, cylc 7 and 8, rose). It can be thought as a replacement for the 
modules used on many high performance computers to load in dependencies (toolchain, NetCDF, etc).

## Prerequisites

First you need to have Apptainer or Singularity installed. If you're running Linux, you're in luck, just follow these [instructions](https://apptainer.org/docs/user/latest/). On Windows, you can use Windows Linux Subsystem (WSL) and on Mac you might have to install Apptainer within a Docker environment. 

## How to build a container

A container needs a definition file, e.g. `conf/umenv_intel.def`. This file lists the operating system, the compilers and the steps to build the libraries. It is a recipe for building the container.

To build the container, type
```
apptainer build umenv_intel2004.sif conf/umenv_intel.def
```
or, on a local laptop if you encounter the error `FATAL: ...permission denied`,
```
sudo apptainer build --force umenv_intel2004.sif conf/umenv_intel.def
```
Now take a cup of coffee.

Note: if you're using Singularity you may replace `apptainer` with `singularity` in the above and below commands.

### Building the container on NeSI

If you don't have access to a Linux system with Apptainer installed, you can also [build the container on Mahuika](https://support.nesi.org.nz/hc/en-gb/articles/6008779241999-Build-an-Apptainer-container-on-a-Milan-compute-node) by submitting the follwowing SLURM job
```
#!/bin/bash -e
#SBATCH --job-name=apptainer_build
#SBATCH --partition=milan
#SBATCH --time=0-08:00:00
#SBATCH --mem=30GB
#SBATCH --cpus-per-task=4

# load environment module
module purge
module load Apptainer

# recent Apptainer modules set APPTAINER_BIND, which typically breaks
# container builds, so unset it here
unset APPTAINER_BIND

# create a build and cache directory on nobackup storage
export APPTAINER_CACHEDIR="/nesi/nobackup/$SLURM_JOB_ACCOUNT/$USER/apptainer_cache"
export APPTAINER_TMPDIR="/nesi/nobackup/$SLURM_JOB_ACCOUNT/$USER/apptainer_tmpdir"
mkdir -p $APPTAINER_CACHEDIR $APPTAINER_TMPDIR
setfacl -b $APPTAINER_TMPDIR

apptainer build --force --fakeroot umenv_intel2004.sif conf/umenv_def.def
```

Once the build completes you will end up with a file `ummenv_intel2004.sif`, which you can copy across platforms.

## How to run a shell within a container

Assuming you have loaded the `Apptainer` module on Mahuika (or have the command `apptainer` available on your system),
```
apptainer shell umenv_intel2004.sif
```
will land you in an environment with compilers
```
Apptainer> 
```
In this environment, you should also have the commands `fcm`, `rose` and `cylc` available.

Note that you may need to bind some directories to access external data inside the container. This is achieved with the `-B` option. For instance,

```
singularity shell -B/scale_wlg_nobackup/filesets/nobackup,/nesi/nobackup,$HOME,/opt/niwa /nesi/nobackup/pletzera/umenv_intel2004.sif
```

## Caching your password

Assuming that you have been given access to `https://code.metoffice.gov.uk`, you can cache your password using
```
Apptainer> source /usr/local/bin/mosrs-setup-gpg-agent
```
Check that your password has been cached with the command
```
Apptainer> rosie hello
https://code.metoffice.gov.uk/rosie/u/hello: Hello alexanderpletzer
```

## Other setup files

You will likely need to set up and edit the following files:
 1. ~/.metomi/rose.conf
 2. ~/.subversion/servers
Please refer to the Metoffice documentation on how to set these files up.

In addition, you'll need the file `$HOME/.metomi/fcm/keyword.cfg`. An example is 
```
location{primary}[nemo] = http://forge.ipsl.jussieu.fr/nemo/svn #/trunk
location{primary}[nemo.xm] = http://forge.ipsl.jussieu.fr/nemo/svn
location{primary}[nemo.x] = http://forge.ipsl.jussieu.fr/nemo/svn

location{primary}[um.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/main
location{primary}[um.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/main/trunk
location{primary}[um.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/main/branches

location{primary}[um_mule.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/mule
location{primary}[um_mule.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/mule/trunk
location{primary}[um_mule.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/mule/branches

location{primary}[um_meta.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/meta

location{primary}[um_meta.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/meta/trunk

location{primary}[um_meta.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/meta/branches

location{primary}[jules.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/jules/main
location{primary}[jules.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/jules/main/trunk
location{primary}[jules.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/jules/main/branches

location{primary}[socrates.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/socrates/main
location{primary}[socrates.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/socrates/main/trunk
location{primary}[socrates.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/socrates/main/branches

location{primary}[um_aux.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/aux

location{primary}[um_aux.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/aux/trunk

location{primary}[um_aux.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/um/aux/branches

location{primary}[ancil.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/main

location{primary}[ancil.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/main/trunk

location{primary}[ancil.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/main/branches

location{primary}[ancil_data.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/data

location{primary}[ancil_data.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/data/trunk

location{primary}[ancil_data.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/data/branches

location{primary}[ancil_contrib.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/contrib

location{primary}[ancil_contrib.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/contrib/trunk

location{primary}[ancil_contrib.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/ancil/contrib/branches

location{primary}[roses-u.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/roses-u

location{primary}[moci.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/moci/main

location{primary}[moci.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/moci/main/trunk

location{primary}[moci.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/moci/main/branches

location{primary}[cice.xm] = file:///nesi/project/uoo03538/um/metoffice-science-repos/cice/main

location{primary}[cice.xm_tr] = file:///nesi/project/uoo03538/um/metoffice-science-repos/cice/main/trunk

location{primary}[cice.xm_br] = file:///nesi/project/uoo03538/um/metoffice-science-repos/cice/main/branches
```

## Building GCOM (UM dependency)

The Unified Model (UM) has additional dependencies, which need to be built as a second step. You will need access to the `code.metoffice.gov.uk` repository.

1. Copy the `umenv_intel2004.sif` file to the the target platform (e.g. Mahuika)
2. `module load Apptainer`
3. `apptainer shell umenv_intel2004.sif`
5. Inside the Apptainer shell type
```
fcm co fcm:gcom.x/branches/dev/andrewpauling/vn6.2_apptainer_port@1344 gcom-1344
cd gcom-1344
rose stem --group=test --name=gcom-1344 --new
```
This will install install GCOM under `~/cylc-run/gcom-1344/share/*/build/lib`.


## Building and running the atmosphere only 

Start a shell inside the container
```
apptainer shell /nesi/nobackup/pletzera/umenv_intel2004.sif
```

Make sure you have the environment variable `UMDIR`, e.g.
```
Apptainer> export UMDIR=/opt/niwa/um_sys/um
```
to point to the location where the input data are stored.

Download the platform configuration
```
Apptainer> source /usr/local/bin/mosrs-setup-gpg-agent
Apptainer> fcm co https://code.metoffice.gov.uk/svn/um/main/branches/dev/andrewpauling/r116716_vn10.7_nesi_apptainer_port
```

Check out the suite, compile and run it
```
Apptainer> rosie checkout u-di148
Apptainer> cd ~/roses/u-di148
```
Update the `config_root_path` and `um_sources` variables in `$HOME/roses/u-di148/app/fcm_make/rose-app.conf`
 to point to the path of your copy. Then type
```
Apptainer> rose suite-run
```

## Building a coupled coupled ocean and atmospheric model

These steps are required when running on Mahuika. First, start by creating file `$HOME/.cylc/flow/global.cylc` with lines:
```
#!Jinja2

# CYLC SITE CONFIGURATION DEFAULTS.

#------------------------------------------------------------------------------
# SSH
# StrictHostKeyChecking=no: avoid the need for an initial manual ssh to
# off-cluster hosts, for the interactive prompt to add it to .ssh/known_hosts.
{% set SSH = "ssh -oBatchMode=yes -oConnectTimeout=8 -oStrictHostKeyChecking=no" %}

#------------------------------------------------------------------------------
# TASK JOB PLATFORMS
[platforms]
    [[mahuika01-background]]
        hosts = mahuika01
        job runner = background
        install target = localhost
        ssh command = {{SSH}}
        [[[meta]]]
             description = "For background jobs on mahuika login node."
             note = "Slurm will run the job where it sees fit."
```

On Mahuika
```
export PYTHONPATH=/opt/niwa/share/bin:/opt/nesi/share/bin:$PYTHONPATH
export APPTAINERENV_PREPEND_PATH=/opt/nesi/share/bin
export APPTAINERENV_PYTHONPATH=$PYTHONPATH 
export CYLC_VERSION=8.1.4
export UMDIR=/nesi/project/uoo03538/um
module purge
module load FCM
module load Apptainer/1.2.5
apptainer shell /nesi/nobackup/pletzera/umenv_intel2004.sif
```

Then, in the container,
```
Apptainer> source /usr/local/bin/mosrs-setup-gpg-agent
Apptainer> rosie co u-di415
Apptainer> cd ~/roses/u-di415
Apptainer> export PROJECT=niwa99999 # Set your project number here! You should have a corresponding entry in ~/.cylc/projects
Apptainer> cylc vip
```




## How to compile an application using the containerised environment

When running `apptainer shell` on Mahuika, directories (`$HOME`, `/nesi/project`, `/nesi/nobackup`) are mounted by default. Additional directories can be mounted with the `-B <dir>` option. 

You can use the compilers inside the container to compile your application. In the example below we compile a simple MPI C application
```
cat > myapp.c << EOF
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    // Initialize the MPI environment
    MPI_Init(NULL, NULL);

export PROJECT=    // Get the number of processes
    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);

    // Get the rank of the process
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

    // Get the name of the processor
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    int name_len;
    MPI_Get_processor_name(processor_name, &name_len);

    // Print off a hello world message
    printf("Hello world from processor %s, rank %d out of %d processors\n",
           processor_name, world_rank, world_size);

    // Finalize the MPI environment.
    MPI_Finalize();
}
EOF
apptainer exec umenv_intel2004.sif mpicc myapp.c -o myapp
```

## Running a containerised MPI application

You can either leverage the MPI inside the application
```
apptainer exec umenv_intel2004.sif mpiexec -n 4 ./myapp
```
or use the hosts's MPI. In the latter case, the MPI versions inside and on the host must be compatible. On NeSI's Mahuika we recommend to use the Intel MPI. In this case your SLURM script would look like:
```
cat > myapp.sl << EOF
#!/bin/bash -e
#SBATCH --job-name=runCase14       # job name (shows up in the queue)
#SBATCH --time=01:00:00       # Walltime (HH:MM:SS)
#SBATCH --hint=nomultithread
#SBATCH --mem-per-cpu=2g             # memory (in MB)
#SBATCH --ntasks=10        # number of tasks (e.g. MPI)
#SBATCH --cpus-per-task=1     # number of cores per task (e.g. OpenMP)
#SBATCH --output=%x-%j.out    # %x and %j are replaced by job name and ID
#SBATCH --error=%x-%j.err
#SBATCH --partition=milan

ml purge
ml Apptainer

module load intel        # load the Intel MPI
export I_MPI_FABRICS=ofi # turn off shm to run on multiple nodes

srun apptainer exec umenv_intel2004.sif ./myapp
EOF
sbatch myapp.sl
```






