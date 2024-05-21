# Snakemake on the supercomputer

```{objectives}
- Understand the different ways Snakemake can be run on clusters
- Evaluate which way suits your needs 
- Make own workflow ready for HPC
```

> Note on versions: These materials assume Snakemake version 8.0 or later. Earlier (before version 8.0) Snakemake versions deal differently with SLURM. Please check the documentation for your version of snakemake. 


## Cluster Snakemake installation

1. Preinstalled and made available via module: `module load snakemake/version`
2. Own Conda/Pip based installation
3. Own container based installation

Always check first if your cluster has the tool available as **module**, then you do not have to worry about installation yourself.
If you need additional packages, you can usually add own packages by `pip install` for your own user or project. It may also be possible to load multiple modules at the same time (feasibility depends on the modules). If that for some reason does not work, you can create your own installation with Conda or Pip, or by making use of containers. **Check your clusters documentation for information on how to do these.**
Ask your friendly cluster support for help or advice, if needed. 


## Ways of running Snakemake on clusters

### Within one SLURM job

#### Interactive

You can run snakemake the same way you would run it on your own computer also on cluster within one interactive or normal batch job (one resource allocation for the whole workflow (= all rules)). The interactive session is is a useful first step to try it out, but does not scale well (only to the resource limits of one interactive job).

In an interactive job (using the interactive partition; some clusters support starting an interactive session with the `sinteractive` command), you would load the snakemake module, and then run `snakemake --cores 1` (or more, if you reserved more) as you would on your local machine.

You can run snakemake in the background for example with [`screen`](https://linux.die.net/man/1/screen).

##### Sbatch script

Another way to run snakemake in **one job** is to use an sbatch script for job submission to run the workflow non-interactive.

```
#!/bin/bash
#SBATCH --job-name=myTest
#SBATCH --account=project_xxxxx
#SBATCH --time=00:10:00
#SBATCH --mem-per-cpu=2G
#SBATCH --partition=test
#SBATCH --cpus-per-task=4

module load snakemake/version
snakemake --cores 4
```

Note how 4 CPUs are reserved ( `#SBATCH --cpus-per-task=4`) and made available to the workflow (`--cores 4`). Within the Snakefile resource distribution and parallelization can be controlled using `threads:X` directive, see the [Snakemake documentation on threads](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#threads).

#### Slurm integration with the [Snakemake SLURM executor plugin](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html)

With the plugin you can define SLURM batch job resources for your snakemake workflow instead of writing an sbatch file.

Snakemake provides a [Snakemake SLURM executor](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html) as plugin, which can be used to let Snakemake run workflows on clusters. For this, you would start an interactive session, from which you could call :

```
module load snakemake/version

snakemake --jobs 1 --executor slurm --default-resources slurm_account=project_xxx slurm_partition=test runtime=10 mem_mb_per_cpu=2000 cpus_per_task=4
```

This command would submit **one new job** (defined by `--jobs 1`) to SLURM for running the whole workflow in that job, similar to the sbatch script above. Instead of defining `cpus_per_task` for the whole workflow, one could also define `threads:X` for each rule.

Other SLURM resource specifications can be found from the table in the [SLURM executor plugin documentation](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html#advanced-resource-specifications).

The SLURM integration also provides the possibility to start multiple SLURM jobs (specified by setting `--jobs` to > 1) for you; for example one for each step. However, with this option you would have to wait in queue for every single step to be executed which generates a lot of overhead, especially for short job steps. Using the Snakemake SLURM integration this way should therefore be avoided in cases where single steps are short. A better solution to scale up your Snakemake workflow (also to multiple nodes) is to use Hyperqueue instead:

### Snakemake at scale using Hyperqueue

[HyperQueue (HQ)](https://it4innovations.github.io/hyperqueue/stable/) is an efficient sub-node task scheduler. Instead of submitting each of your computational tasks as separate Slurm jobs or job steps, you can allocate a large resource block and then use HyperQueue to submit your tasks to this allocation, even when running your workflow on multiple nodes.
The [generic cluster executor](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/cluster-generic.html) plugin can be used to configure hyperqueue runs, e.g. `snakemake --executor cluster-generic --cluster-generic-submit-cmd "hq submit ..." `.

- Useful for high-throughput jobs (many steps and rules on many files, needing a lot of resources)
- Extra benefit: Multiple nodes can be used under the same job allocation

For more information, please check out the provided Hyperqueue example.


## Config/Profile files

Snakemake takes many parameters as command line arguments/flags. However, when working with clusters these can become quite many very fast. Many of which also do not change very often, e.g. `slurm-account`. One way to make the command line call shorter, is to define some of the parameters in [**profiles**](https://snakemake.readthedocs.io/en/stable/executing/cli.html#profiles). These can be global or workflow specific. 

A profile is written in `YAML format` and usually called `config.yml` and can be provided to Snakemake using the `--profile` flag (Note to only provide the path where Snakemake should look for the profile file, e.g. `profiles/slurm/` (without the profile filename)). An option defined by `--someoption` in the Snakemake command line call becomes `someoption: ` in the profile. Example profile:

```
jobs: 1
default-resources: 
  slurm_account: project_xxx
```

Check out another [Snakemake profile example from the Tuesdays Tools and Techniques course](https://coderefinery.github.io/TTT4HPC_parallel_workflows/parallelization/parallelize_using_workflow_manager/#create-and-run-snakemake-workflow).

## Good practices

- General: 
    - As with all code, it is advised to also keep your Snakefiles under **version control**, e.g. using `git`
    - Consider **containerizing** your whole workflow for portability
- When running Snakemake on clusters: 
    - Try to **avoid unnecessary read and write operations**, check if use of fast local disk is possible
    - Consider summarizing small jobs/job steps into one job  when you want to use the SLURM plugin to limit SLURM accounting overhead, e.g. by [job grouping](https://snakemake.readthedocs.io/en/stable/executing/grouping.html)
    - Try to avoid creating a lot of files, especially in the same folder, [Snakemake can remove temporary files after the job is finnished](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#protected-and-temporary-files).

##  Cluster specific information

`````{tabs}
  ````{group-tab} CSC Puhti or LUMI
Running Snakemake on CSC's Puhti or LUMI: [Slides](https://a3s.fi/CSC_training/snakemake_hackathon.html#/snakemake-hackathon-in-csc-supercomputers).
  ````
  ````{group-tab} UPPMAX
TBD
  ````
`````


```{keypoints}
- Use the `snakemake module` if your cluster provides one; add own packages or modules where necessary
- If that fails, consider creating your own environment; follow cluster specific instructions
- Use interactive jobs to test and setup your Snakemake workflow
- Use normal batch jobs for running your tested workflows via sbatch script or using the Snakemake SLURM executor (`--jobs = 1`)
- Use Hyperqueue to scale your workflow on the cluster, especially when you need multiple nodes
```
