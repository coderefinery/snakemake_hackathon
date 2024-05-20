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

We can also use named inputs and outputs, and refer to them using their names:
```bash
rule sorter:
    input:
        a="hello.txt", 
        b="hello2.txt"
    output:
        "sorted_greetings.txt",
    shell:
        "sort {input.a} {input.b} > {output}"
```

### Wildcards
In the examples above we used hard-coded filenames to tell the input or output values. To make the workflows more generalized, we can use wildcards. Wildcards are variables that are replacing the actual filenames or any other value, like for example a path. Snakemake resolves them automatically based on either the target file or other input/outputs in a Snakefile. Let's add wildcards to the copy example, so it will work for any text file:
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

### Python code in Snakefile

Another powerful feature is the possibility to add Python code and function inside of Snakefile. We also have access to Python libraries, by using traditional `import` keyword. For example, the following code searches for all text files in the given directory, and appends their content in the `copy.txt` file.
```Python
import os

def find_txt_files(path):
    txt_files = [file for file in os.listdir(path) if file.endswith(".txt")]
    return txt_files

rule append:
    input:
	    find_txt_files(".")  # . means the current working directory
    output: 
	    "results/copy.txt"
    shell:
	    "cat {input} >> {output}"
```

How is this different than wildcards? Here, we did not resolve or specified any file. We can simply call `snakemake --cores 1` without any arguments. This workflow searches for the input files by itself, without any guidance.

Another way of using Python is replacing the shell command with Python script:
```Python
rule print_content:
    input:
	    "hello.txt"
    run:
        with open(input[0]) as in_file:
            for l in in_file:
                print(l)
```

This approach is fine for small code snippets. For bigger scripts, it is better to move code to a separate file and call it in the `Snakefile`. Let's create the `read_and_print.py` file:
```Python
with open(snakemake.input[0]) as in_file:
    for l in in_file:
        print(l)
```
and modify the `Snakefile`:
```bash
rule print_content:
    input:
	    "hello.txt"
    script:
        "read_and_print.py"
```
In the Python file, we have access to the `snakemake` object, which allows us to get access to input and output files! We do not need to parse any command line arguments. The access is granted because of the `script` keyword, which works like a wrapper around the script. Similar integrations are available for other languages as well: R, Markdown, Julia, Rust, Bash and Jupyter Notebook. See the current list in the [docs](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#external-scripts)


### Dependencies between rules
So far, we only examined `Snakefile`s with only one rule. Let's have a look at a bigger workflow.
```bash
rule concatenate:
    input:
        expand("data/file{n}.txt", n=[1, 2, 3])
    output:
        "results/concatenated.txt"
    shell:
        "cat {input} > {output}"

rule count_words:
    input:
        "results/concatenated.txt"
    output:
        "results/word_count.txt"
    shell:
        "wc -w {input} > {output}"
    default_target: True
```
This workflow concatenates three input files (`data/file1`, `data/file2`, `data/file3`) and then counts how many words there are. The `Snakefile` consists of three rules. The dependencies between rules are defined by the input and output filenames. The rule `concatenate` generates the output file: `results/concatenated.txt`, which then is used as a input to the `count_words` rule. 

Snakemake uses top-down approach to automatically resolve the dependencies between rules. That means, Snakemake will start from the last rule (so-called target rule) and go backwards, searching for the input needed to execute that step. By default, the target rule is the first rule in the Snakefile. In this case, we want to run the `count_words` rule as the target (first we concatenate, then we count the words). We have to somehow tell Snakemake what is the final output of this workflow. To achieve that, a common practice is adding a mock `rule all` at the beginning of the `Snakefile` that encapsulates the final outputs of the workflow:
```bash
rule all:
    input:
        "results/word_count.txt"

rule concatenate:
    input:
        expand("data/file{n}.txt", n=[1, 2, 3])
    output:
        "results/concatenated.txt"
    shell:
        "cat {input} > {output}"

rule count_words:
    input:
        "results/concatenated.txt"
    output:
        "results/word_count.txt"
    shell:
        "wc -w {input} > {output}"
```

Snakemake will see that we want the `results/word_count.txt` file, and will search in the `Snakefile` to see what rule produces this exact output. Then, it will repeat this process recursively until all dependencies are resolved. By doing so, so-called directed acyclic graph (DAG) is created. It is a step-by-step execution plan for Snakemake. We can see and analyze this graph by calling Snakemake with `--dag` flag:
```bash
snakemake --cores 1 --dag
```
The result can be visualized using either the `GraphViz` package, or online on [this site](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://dreampuf.github.io/GraphvizOnline/&ved=2ahUKEwjLk6SBqJyGAxVCLRAIHVNWBZMQFnoECBQQAQ&usg=AOvVaw2Sw6OnaIb_oZkOtu44VcNz).
```bash
snakemake --cores 1 --dag | dot -Tpng > dag.png
```

One improvement to this `Snakefile` would be to directly refer to the outputs of the rules by using the rule's name. This approach is more robust, as we can freely change the output names, without breaking the workflow. Remember that you can only refer to the rules that were defined before, so we cannot modify in that way the `all` rule)! 
```bash
rule all:
    input:
        "results/word_count.txt"

rule concatenate:
    input:
        expand("data/file{n}.txt", n=[1, 2, 3])
    output:
        "results/concatenated.txt"
    shell:
        "cat {input} > {output}"

rule count_words:
    input:
        rules.concatenate.output
    output:
        "results/word_count.txt"
    shell:
        "wc -w {input} > {output}"
```

### Force execution

Snakemake is designed to efficiently manage the execution of complex workflows by only computing the parts that are necessary. Specifically, it checks the presence and timestamps of the output files specified in your workflow rules. If an output file already exists and is up to date, Snakemake will skip the computation steps that produce that file, saving time and resources by avoiding redundant computations.

However, there are situations where you might want to recompute the entire workflow, regardless of the existing files. In such cases, you can force Snakemake to recompute all the steps by using the `-F` or `--forceall` flag. This tells Snakemake to ignore the existing output files and re-run all the rules as if the files were missing.

```bash
snakemake --cores 1 -F
```

### Dependencies of steps

DAG: Directed acyclic graphs are built when Snakemake is run

- implicit parallelization (uses number of cores given to run tasks in parallel)
- Resolves step dependencies before execution

### Parallelization

`threads`
`--cores`



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
