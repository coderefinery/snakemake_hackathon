# Snakemake introduction

Snakemake is a powerful workflow management system designed to create reproducible and scalable data analyses. Originating in the bioinformatics community, Snakemake has gained widespread use in various scientific and data-intensive fields. It allows users to define workflows in a simple, readable, and maintainable way using a Python-based language. With built-in support for parallel execution and resource management, Snakemake can efficiently handle workflows on various scales, from small-scale local computations to large-scale distributed computing environments.

```{objectives}

- Understand the components and features of a Snakefile
- Understand portability and reproducibility in the context of Snakemake
- Write a snakefile
- Run Snakemake from the shell
- Understand the key features of Snakemake
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

### Installation

To install Snakemake, we can use Python Package Installer (`pip`). First, ensure you have Python installed on your system.

```bash
pip install snakemake
```

To verify the installation, run:
```bash
snakemake -v
```

You should be able to see the version of Snakemake installed on your system.

### Snakefile

We can define Snakemake workflows using files called `Snakefile`s. Workflow are defined in terms of rules. Each rule describes input files, outputs and a command or script to be run. Let's have a look at a simple, one step workflow, that will copy the content of one file into another.

```bash
rule copy:
    input:
	    "hello.txt"
    output: 
	    "hello_copy.txt"
    shell:
	    "cp {input} {output}"
```

This Snakefile defines one input file `hello.txt`, one output file `hello_copy.txt` and a shell command `cp {input} {output}`. The shell command is formatted automatically by Snakemake. The `input` and `output` are substituted using the values provided in the previous section.

Before running, copy the content of the `Snakefile` into the file called `Snakefile` (without an extension) and create `hello.txt` file with some content:
```bash
echo "Hello Snakemake!" > hello.txt
```

To run the `Snakefile`, we run the following command:
```bash
snakemake --cores 1
```

Snakemake executes the `Snakefile` with the one rule in it. The content is copied! Note two things: the `--cores` flag and that we did not specified the `Snakefile`. By default, Snakemake will search for a `Snakefile` in the current directory. We can specify which `Snakefile` to run using the `-s` flag. For example:
```bash
snakemake --cores 1 -s Snakefile_2
```

will run the `Snakefile_2`. Thanks to that, now we can also how multiple `Snakefile`s in one directory. The `--cores` flag tells Snakemake how many resources (cores) from our computer Snakemake can use to execute the workflow. It is mandatory flag, so one cannot omit it.


- Definition of the workflow via rules
- File based approach
- `expand`

### Rules

A rule do not necessarily have to have only one input or output. If so, we can use indices to refer to each input/output value:

```bash
rule sorter:
    input:
        "hello.txt", 
        "hello2.txt"
    output:
        "sorted_greetings.txt",
    shell:
        "sort {input[0]} {input[1]} > {output}"
```

In the example above (and the one before) we used hard-coded filenames to tell the input or output values. To make the workflows more generalized, we can use wildcards. Wildcards are variables that are replacing the actual filenames or any other value, like for example a path. Snakemake resolves them automatically based on either the target file or other input/outputs in a Snakefile. Let's add wildcards to the copy example, so it will work for any text file:
```bash
rule copy:
    input:
	    "{data}.txt"
    output: 
	    "{data}_copy.txt"
    shell:
	    "cp {input} {output}"
```

To run it, we have to tell Snakemake the value of the `data` wildcard. We can do that, by specifying what kind of output file we want to produce. In this case, we want to have `hello_copy.txt` - the copy of the `hello.txt` file.
```bash
snakemake --cores 1 hello_copy.txt
```

Snakemake parses the `hello_copy.txt` argument and detects that the wildcard is equal to `hello`. Then, Snakemake looks for the `hello.txt` file. 


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
