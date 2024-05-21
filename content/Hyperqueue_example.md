# Hyperqueue example: Snakemake at Scale 

This example is adapted from ["Using CSC Computing Environment Efficiently"](https://csc-training.github.io/csc-env-eff/) course and the [official Snakemake tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/basics.html).

## Use HyperQueue Executor to Submit Jobs

If you have many short processes, it's advisable to use HyperQueue instead of Snakemake SLURM integration to improve throughput and decrease 
load on the system batch scheduler.

You can load HyperQueue and Snakemake modules on different clusters as described below:

`````{tabs}
  ````{group-tab} CSC Puhti
    ```bash
    module load hyperqueue/0.16.0
    module load snakemake/8.4.6
    ```
  ````
  ````{group-tab} LUMI
    ```bash
    module use /appl/local/csc/modulefiles/
    module load hyperqueue/0.18.0
    module load snakemake/8.4.6
    ```
  ````
  ````{group-tab} Other
    ```bash
    module load hyperqueue
    module load snakemake
    ```
    If available, otherwise ask your cluster admin to install it for you.
  ````
`````

One can use HyperQueue executor settings depending on the Snakemake version as below:

`````{tabs}
  ````{group-tab} version 8.x.x
    `snakemake --executor cluster-generic --cluster-generic-submit-cmd "hq submit ..."  `
  ````
  ````{group-tab} version 7.x.x
    `snakemake  --cluster "hq submit  ..." `
  ````
`````


## Submit Snakemake Workflow on Cluster

Download tutorial materials (scripts and data), which have been adapted from the 
[official Snakemake documentation](https://snakemake.readthedocs.io/en/v6.6.1/executor_tutorial/google_lifesciences.html), 
from CSC Allas object storage as below (no login required):

```bash
wget https://a3s.fi/snakemake_scale/snakemake_scaling.tar.gz
tar -xavf snakemake_scaling.tar.gz
```

The downloaded material includes scripts and data to run snakemake pipeline:

- **data** folder includes samples and other data needed to run this example 
- **image** folder includes information about the computational environment needed to run the example
  - a container image, container definition file and a conda environment file
- **scripts** includes a script that will be run as part of the workflow, in addition to other command line tools  
- **Snakefile**  with all rules defining the workflow
- **snakemake_hq_lumi.sh** for running the example on LUMI
- **snakemake_hq_puhti.sh** for running the example on Puhti

To run the example on UPPMAX and others, please use the Puhti batch job script.

The computing environment for running the job is provided as container image (`image/tutorial.sif`), you can check the container definition file `image/tutorial.def` and the conda environment `image/tutorial.yaml` file for the contents of the image.

The Snakefile describes a workflow from bioinformatics and is described in detail in the [official Snakemake tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/basics.html).

An example `snakemake_hq_puhti.sh` content is posted below:

```bash 
#!/bin/bash
#SBATCH --job-name=snakemake
#SBATCH --account=<project>  # replace <project> with your project, e.g. for CSC: project_2001234
#SBATCH --partition=small
#SBATCH --time=00:10:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=40
#SBATCH --mem-per-cpu=2G

module load hyperqueue/0.16.0
module load snakemake/8.4.6

# Specify a location for the HyperQueue server
export HQ_SERVER_DIR=${PWD}/hq-server-${SLURM_JOB_ID}
mkdir -p "${HQ_SERVER_DIR}"
 
# Start the server in the background (&) and wait until it has started
hq server start &
until hq job list &>/dev/null ; do sleep 1 ; done
 
# Start the workers in the background and wait for them to start
srun --exact --cpu-bind=none --mpi=none hq worker start --cpus=${SLURM_CPUS_PER_TASK} &

hq worker wait "${SLURM_NTASKS}"

# `--use-singularity` tells snakemake to make use of the singularity directive in the snakefile (line 4)
snakemake -s Snakefile --jobs 1 --use-singularity --executor cluster-generic --cluster-generic-submit-cmd "hq submit --cpus 5"

# for snakemake versions 7.x.xx, use command: snakemake -s Snakefile --jobs 1 --use-singularity --cluster "hq submit --cpus 2"

# Wait for all jobs to finish, then shut down the workers and server
hq job wait all
hq worker stop all
hq server stop

```

The default script provided above is not optimised to run in high-throughput way as snakemake workflow manager just submits **one job at a time** (defined as `--jobs 1`) to the hyperqueue scheduler. You can parallelise workflow tasks (i.e., rules in snakemake) by submitting more jobs snakemake as below:

`snakemake -s Snakefile --jobs 8 --use-singularity --executor cluster-generic --cluster-generic-submit-cmd "hq submit --cpus 5"`

Correct this modification (exchanging 1 with 8 for number of jobs) in the batch script and run the batch script with `sbatch snakemake_hq_puhti.sh`.

One can also use more than one node to achieve even more high-throughput as HyperQueue can make use of multi-node resource allocations.

Please note that just increasing the number jobs will not alone automatically run all those jobs at the same time. 
*Jobs* parameter from *snakemake* is just a maximum limit for concurrent jobs. Jobs will be eventually run when resources are available. 
In our case we will have up to 8 parallel steps running, each taking 5 CPUs using all of the reserved 40 CPUs in batch script. 
In practice, it is a good idea to dedicate few CPUs for the workflow manager itself. 


## Follow the progress of jobs

You can already check the progress of your job by simply observing the current folder where you can see lot of new 
task-specific folders are being created. However, there are formal ways to check the progress of your jobs as shown below:


`````{tabs}
  ````{group-tab} Full job monitoring
    ```bash
    squeue -j <slurmjobid>
    # or
    squeue --me
    # or
    squeue -u $USER
    ```
  ```` 
  ````{group-tab} Sub task monitoring
    ```bash
    module load hyperqueue
    export HQ_SERVER_DIR=$PWD/hq-server-<slurmjobid>
    hq worker list   
    hq job list
    hq job info <hqjobid>
    hq job progress <hqjobid>
    hq task list <hqjobid>
    hq task info <hqjobid> <hqtaskid>
    ```
  ````
````` 

### How do you clean different task-specific folders automatically?

HyperQueue creates task-specific folders (i.e., job-`<n>`) in the same directory from where you have submitted batch script.
These are sometimes useful for debugging. However if your code is working fine, the creation of such large number
folders may be annoying besides causing some overhead to parallel file systems like Lustre. You can prevent creating
such task-specific folders by setting `stdout` and `stderr` flags to `none` as shown below:

```bash
snakemake -s Snakefile -j 24 --use-singularity --executor cluster-generic --cluster-generic-submit-cmd "hq submit --stdout=none --stderr=none --cpus 5 "
```

## More Information

- [Hyperqueue documentation](https://it4innovations.github.io/hyperqueue/stable/)
- [Snakemake documentation](https://snakemake.readthedocs.io/en/stable/)

