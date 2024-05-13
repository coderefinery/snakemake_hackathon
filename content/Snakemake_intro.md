# Snakemake introduction

```{objectives}
- TBD
```

## What is a workflow?

- Set of computational steps
- Step defined by input, process, output
- Connection through input and output files

## Why workflow tools?

Example usecase from GUI -> multiple scripts run manually -> bash script to run all 

> Issues with bashscripts: running only parts (commenting/uncommenting), running only for some files 

## Workflow tools

- Many languages to define a workflow
- "Framework" for executing a workflow
- Heavily used in bioinformatics

reproducible description of workflow, "smart"

## Snakemake

Why Snakemake?

- Available as Python package (pip, conda), but not python specific
- Python based syntax (easy read)
- Possible to extend with embedded Python code, but not necessary 
- Snakemake workflow catalog

### Snakefile

- Definition of the workflow via rules
- File based approach
- `expand`

### Rules

```Python
rule copy_file:
    input:
	"input.txt"
    output: 
	"output.txt"
    shell:
	"cp {input} {output}"
```

- Expand
- localrules?



### Execution

Local runner/executor: `snakemake --cores 1`

- command line arguments vs config file

### Dependencies of steps

DAG: Directed acyclic graphs are built when Snakemake is run

- implicit parallelization (uses number of cores given to run tasks in parallel)
- Resolves step dependencies before execution

### Parallelization

`threads`
`--cores`


### Wildcards

...

### Python and Snakemake

- Snakemake object can be used in Python scripts
- Custom Python functions

### Error recovery and re-entry

- `retries: X`
- Keep going
- Resume after failure


### Portability

#### Conda

#### Containers

## Provenance features

(from snakemake docs:)
The ability to track the provenance of each generated result is an important step towards reproducible analyses. Apart from the report functionality discussed before, Snakemake can summarize various provenance information for all output files of the workflow. The flag --summary prints a table associating each output file with the rule used to generate it, the creation date and optionally the version of the tool used for creation is provided. Further, the table informs about updated input files and changes to the source code of the rule after creation of the output file. 

## Reporting

Basic report on run in terminal/ in file

HTML report available after execution to provide statistics

Monitoring availabe via panoptes (external tool)
- during execution
- live view and status
 
## Interoperability

Workflows can be exported to eg Common Workflow Language (CWL) to be executed with other tools too (eg Cromwell, Toil ?) 

Notes from here: https://snakemake.readthedocs.io/en/stable/executing/interoperability.html#cwl-export

## Unit tests

Generate automatically with Snakemake.

## Workflow reproducibility

## Visualization

Example how to viz the workflow: https://coderefinery.github.io/reproducible-research/workflow-management/#visualizing-the-workflo## Visualization

Example how to viz the workflow: https://coderefinery.github.io/reproducible-research/workflow-management/#visualizing-the-workfloww

```{keypoints}
- TBD
```
