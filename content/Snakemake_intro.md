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

```{figure} img/workflow.png
:width: 60%
```

## Workflow tools and why we need them?

Imagine having some data processing workflow that takes raw data, converts and filters it, and finally produces some result, for example a plot. Each step of the workflow is done using command line tools and scripts written in different languages. You could run everything manually: step by step, checking each output and passing it to the next script or tool. 

Soon enough, you may start automating your work. The simplest way to automate would be to use bash script (if you use MacOS/Linux). Your script may look like this:
```bash
wget http://example.com/dataset_2024.zip
unzip dataset_2024.zip
python filter.py dataset_2024.txt filtered_dataset_2024.txt
python transform.py filtered_dataset_2024.txt transformed_dataset_2024.txt
Rscript stats.r filtered_dataset_2024.txt statistics.csv
Rscript plot.r statistics.csv plot.png
```

Although it works, this solution has some problems: 
- How to quickly compute the results for the dataset from 2023? 
- How to only run the `stats.r` script for unfiltered data? 
- Have you noticed that I made a typo in the bash script and did not use transformed data for the `stats.r` script? 
- How to compute the results for all datasets available (there are 100 of them) using a computer with 20 cores?

As your workflows grow in terms of number of steps, or also the size of dataset, managing this workflow might become more and more problematic. Fortunately, over the years many solutions arose to manage and automate workflow execution. They offer plenty of features, supporting different stages of workflows: starting with the definition of a workflow, executing it and finally, gathering and sharing the results.

> By supporting the top layer, a workflow management system can promote the center layer, and thereby help to obtain true sustainability.

```{figure} img/sustainability.jpeg
:width: 60%
:alt: Taken from Mölder F, Jablonski KP, Letcher B, Hall MB, Tomkins-Tinch CH, Sochat V, Forster J, Lee S, Twardziok SO, Kanitz A, Wilm A, Holtgrewe M, Rahmann S, Nahnsen S, Köster J. Sustainable data analysis with Snakemake. F1000Res. 2021 Apr 19;10:33. doi: 10.12688/f1000research.29032.2. PMCID: PMC8114187.
```

## Why Snakemake?

- Highly popular tool: 11 new citations per week and over 1,000,000 downloads
- Available as Python package (pip, conda), but not Python specific!
- Python-based syntax (easy to read, quick to develop)
- Possible to extend with embedded Python code, but not necessary 
- Supports multiple different scripting languages and all command line tools
- Plenty of useful features for automating the work

It is important to remember that Snakemake does not require Python knowledge or limits you to using Python code.  

### Installation

To install Snakemake on your own computer, we can use Python Package Installer (`pip`). First, ensure you have Python installed on your system.

```bash
pip install snakemake
```

To verify the installation, run:
```bash
snakemake -v
```

You should be able to see the version of Snakemake installed on your system.

### Snakefile

We can define Snakemake workflows using so-called `Snakefile`s. Workflows are defined in terms of rules. Each rule describes input files, outputs and a command or script to be run. Let's have a look at a simple, one step workflow, that will copy the content of one file into another.

```bash
rule copy:
    input:
	    "hello.txt"
    output: 
	    "hello_copy.txt"
    shell:
	    "cp {input} {output}"
```

This `Snakefile` defines one input file `hello.txt`, one output file `hello_copy.txt` and a shell command `cp {input} {output}`. The shell command is formatted automatically by Snakemake. On runtime, `{input}` and `{output}` are substituted by the filenames provided.

To run this example, copy the above content into a file called `Snakefile` (without an extension) and create a `hello.txt` file in the same folder with some content using your favorite editor or in Linux/Mac you can also do:
```bash
echo "Hello Snakemake!" > hello.txt
```

To run the `Snakefile`, we run the following command:
```bash
snakemake --cores 1
```

Snakemake executes the `Snakefile` with the one rule in it. The content is copied! Note two things: the `--cores` flag and that we did not specified the `Snakefile`. By default, Snakemake will search for a `Snakefile` in the current directory. We can specify which `Snakefile` to run using the `--snakefile` flag. For example:
```bash
snakemake --cores 1 --snakefile Snakefile_2
```

will run the `Snakefile_2`. Thanks to that, now we can also have multiple `Snakefile`s in one directory. The `--cores` flag tells Snakemake how many resources (cores) from our computer Snakemake can use to execute the workflow. It is a mandatory flag, so one cannot omit it. You may however also use `--jobs` instead of `--cores` which does not make a difference on a local computer.

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
In the examples above we used hard-coded filenames to define the input and output values. This is not feasible for most workflows dealing with more than a few files. To generalize the workflows, we can use wildcards. Wildcards are variables that replace the actual filenames or any other value, like for example a path. Snakemake resolves them automatically based on either the target file or other input/outputs in the Snakefile. Let's add wildcards to the copy example, so it will work for any text file:
```bash
rule copy:
    input:
	    "{data}.txt"
    output: 
	    "{data}_copy.txt"
    shell:
	    "cp {input} {output}"
```

To run it, we have to tell Snakemake the value of the `data` wildcard. We can do that, by specifying what kind of output /target file we want to produce. In this case, we want to have `hello_copy.txt` - the copy of the `hello.txt` file.
```bash
snakemake --cores 1 hello_copy.txt
```

Snakemake parses the `hello_copy.txt` argument and detects that the wildcard is equal to `hello`. Then, Snakemake looks for the `hello.txt` file.

### Python code in Snakefile

Another powerful feature is the possibility to add Python code and function inside of Snakefile. We also have access to Python libraries, by using traditional `import` keyword. For example, the following code searches for all text files in the given directory, and appends their content in the `copy.txt` file.
```bash
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

How is this different than wildcards? Here, we did not resolve or specify any filename. We can call `snakemake --cores 1` without any arguments. This workflow searches for the input files by itself, without any guidance. Use with care, and for example test that the files that are found are the ones you actually want to process by doing a dry-run, using `--dry-run` when calling Snakemake.

Another way of using Python is replacing the shell command with Python script:
```bash
rule copy:
    input:
	    "hello.txt"
    output:
        "hello_copy.txt"
    run:
        with open(input[0]) as in_f, open(output[0], "w") as out_f:
            for l in in_f:
                out_f.write(l)
```

This approach is fine for small code snippets. For bigger scripts, it is better to move code to a separate file and call it in the `Snakefile`. Let's create the `copy.py` file:
```Python
with open(snakemake.input[0]) as in_f, open(snakemake.output[0], "w") as out_f:
    for l in in_f:
        out_f.write(l)
```
and modify the `Snakefile`:
```bash
rule copy:
    input:
        "hello.txt"
    output:
        "hello_copy.txt"
    script:
        "copy.py"
```
In the Python file, we have access to the `snakemake` object, which allows us to get access to the content of the `input` and `output` directives! We do not need to parse any command line arguments. The access is granted by the `script` keyword, which works like a wrapper around the script. Similar integrations are available for other languages as well: R, Markdown, Julia, Rust, Bash and Jupyter Notebook. See the current list in the [Snakemake documentation](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#external-scripts).


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
```
This workflow concatenates three input files (`data/file1.txt`, `data/file2.txt`, `data/file3.txt` using `expand`, another way to define e.g. filenames more specific than wildcards) and then counts how many words there are in the files using the command line tool `wc`. The `Snakefile` consists of three rules. The dependencies between rules are set by the input and output filenames. The rule `concatenate` generates the output file: `results/concatenated.txt`, which then is used as an input to the `count_words` rule. 

Snakemake uses top-down approach to automatically resolve the dependencies between rules. That means, Snakemake will start from the last rule (so-called `target rule`) and go backwards, searching for the input needed to execute that step. By default, the target rule is the first rule in the Snakefile. In this case, we want to run the `count_words` rule as the target (first we concatenate, then we count the words). We have to somehow tell Snakemake what is the final output of this workflow. To achieve that, a common practice is adding a mock `rule all` at the beginning of the `Snakefile` that encapsulates the final outputs of the workflow:
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

Snakemake will see that we want the `results/word_count.txt` file, and will search in the `Snakefile` to see what rule produces this exact output. Then, it will repeat this process recursively until all dependencies are resolved. By doing so, a so-called directed acyclic graph (DAG) is created. It is a step-by-step execution plan for Snakemake. We can see and analyze this graph by calling Snakemake with `--dag` flag:
```bash
snakemake --cores 1 --dag
```
The result can be visualized using either the `GraphViz` package (which needs to be installed separately, see example call below), or using [GraphVizOnline](https://dreampuf.github.io/GraphvizOnline/).
```bash
snakemake --cores 1 --dag | dot -Tpng > dag.png
```

One improvement to this `Snakefile` would be to directly refer to the outputs of the rules by using the rule's name. This approach is more robust, as we can freely change the output names, without breaking the workflow. Remember that you can only refer to the rules that were defined before, so we cannot modify in that way the `all` rule! 
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

Snakemake is designed to efficiently manage the execution of complex workflows by only computing the parts that are necessary. Specifically, it checks the presence and timestamps from the metadata of the output files specified in your workflow rules. If an output file already exists and is up to date, Snakemake will skip the computation steps that produce that file, saving time and resources by avoiding redundant computations.

However, there are situations where you might want to recompute the entire workflow, regardless of the existing files. In such cases, you can force Snakemake to recompute all the steps by using the `-F` or `--forceall` flag. This tells Snakemake to ignore the existing output files and re-run all the rules as if the files were missing.

```bash
snakemake --cores 1 --forceall
```

Besides forced execution, Snakemake may execute a rule again in two more cases: if the rule definition has changed since the last execution (for example by adding a new output file) or if the script that is executed has changed (only when using the `script` keyword). 

### Parallelization

So far, we only used Snakemake with one core (`--cores 1`). Let's say we have a workflow that processes four files. Each file is processed with two steps: `modify_file` and `count_words`. 

The rule `modify_file` copies the content of the input file to the outputfile using `cat` and redirection `>`, then appends the content of the params directive to the output file using `echo` and redirect append `>>`. In the end the process pauses for 5 seconds using `sleep 5`.

The rule `count_words` uses the command line tool `wc` with the argument `-w` to count the number of words in the input file and writes the number to the output file. In the end the process pauses for 5 seconds using `sleep 5`.
```bash
rule all:
    input:
        "results/word_count_file1.txt",
        "results/word_count_file2.txt",
        "results/word_count_file3.txt",
        "results/word_count_file4.txt",

rule modify_file:
    params:
        msg="This was modified by Snakemake!"
    input:
        "data/{file}.txt"
    output:
        "results/modified_{file}.txt",
    shell:
        "cat {input} > {output} && echo '{params.msg}' >> {output}  && sleep 5"

rule count_words:
    input:
        "results/modified_{file}.txt",
    output:
        "results/word_count_{file}.txt",
    shell:
        "wc -w {input} > {output} && sleep 5"
```

When we create a DAG for that workflow, we see that there are four branches in the graph - one for each file. 

```{figure} img/parallel_workflow.png
:width: 60%
```

If we run Snakemake with the following command:
```bash
snakemake --cores 1
```

The execution will take a little bit more than 40 seconds. Snakemake uses only one core, executing one job at a time. Let's increase number of cores to 2. To run the full workflow again, even though the files have already been processed, remember to add the `--forceall` flag.
```bash
snakemake --cores 2 --forceall
```
Now we can see in the terminal output, that Snakemake is running two jobs at a time, using two cores. The execution time was around two times faster. Snakemake automatically detected, that some parts of the workflow can be run in parallel, and used the provided resources to parallelize the work. In this case, each file can be processed independently, there is no aggregation or combining the results.


### Error recovery and re-entry

Snakemake offers error recovery features. Analyzing the above example, what would happen if one of the input files that we try to modify is corrupted, i.e. not readable? Snakemake will automatically stop the execution of the entire workflow, even though three other files can be processed correctly. Although this behaviour may be useful in some cases, sometimes we may want to compute as many results as we can. To enable this, we can use the `--keep-going` flag. It makes Snakemake keep going as far into workflow as it possibly can, computing all results that do not rely on corrupted files or failed jobs.
```bash
snakemake --cores 2 --keep-going
```

Another error recovery strategy is retrying, which is useful for example when your workflow relies on some online resource.
```bash
rule get_data_from_server:
    output:
        "test.txt"
    retries: 3
    shell:
        "curl https://some.unreliable.server/test.txt > {output}"
```

In cases where a server does not respond with any data, the `curl` command exists with an error. Snakemake quickly retries the job three times. Retrying can be set globally for all jobs using the `--retries` flag followed by the number of retries. 

## Reporting

Snakemake generates execution logs with the content that is also printed in the terminal and stores them in the (hidden) `.snakemake` directory located in the directory where the `snakemake` command is executed . After executing a workflow, Snakemake saves metadata about the execution in that directory. To generate a proper report, we use the `--report` flag when executing the `snakemake` command followed by the filename for the report.

Reports in Snakemake are `html` documents that are generated after workflow execution. They summarize the work done by Snakemake together with some statistics. To generate `html` reports, an extra Python package called `pygments` needs to be installed, for example using `pip`:
```bash
pip install pygments
```
Let's see how we can generate a report. For that, we can use one of the previous workflows.
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

```bash
snakemake --report report.html
```

We can add more information to the report, including the output data of our workflow. In such cases, data is added to the HTML document and can be downloaded. This way we can share the results (at least if they are quite small) with the report and execution information!
```bash
rule all:
    input:
        "results/word_count.txt"

rule concatenate:
    input:
        expand("data/file{n}.txt", n=[1, 2, 3])
    output:
        report(
            "results/concatenated.txt",
            category="Step 1"
        )
    shell:
        "cat {input} > {output}"

rule count_words:
    input:
        "results/concatenated.txt"
    output:
        report(
            "results/word_count.txt",
            category="Step 2"
        )
    shell:
        "wc -w {input} > {output}"

```
By adding the `report()` around our output files, we include both output text files in the report. When opening the report in a web browser, we can now also access the output files and download them.


## Monitoring

> Warning: this section may not work for Windows users!

Snakemake also offers an option for monitoring the workflow from outside of the current workflow execution using an extra tool called `panoptes`. It provides a simple web page with an API to which Snakemake connects. For Linux and Mac, `panoptes` can be installed using `pip`:
```bash
pip install panoptes-ui
```

Panoptes will run a server on `localhost` (i.e. on our computer), which we can start using the following command:
```bash
panoptes
```
To close the server, we can use `CTRL+C` in the terminal. The server is running by default on `http://127.0.0.1:5000`. If we open our browser and visit this website, we will see the graphical interface for monitoring workflows. Note that now we cannot use our terminal, because it is used by `panoptes`. We have to open a new one to be able to run new commands. To connect Snakemake to the monitoring, we modify our `snakemake` command:
```bash
snakemake --cores 1 --wms-monitor http://127.0.0.1:5000
```
If you server has a different URL, replace it in the command above. When we execute the workflow, we can see some basic statistics about the execution.

Warning: this set-up might be not secure enough for the production use. See [this thread](https://github.com/panoptes-organization/panoptes/issues/172) for more details.


## Workflow reproducibility

Reproducibility in computational workflows ensures that an analysis can be consistently repeated with the same results, which is crucial for validating findings, ensuring transparency, and facilitating collaboration. It allows other researchers to verify results, understand methodologies, and build upon previous work.

### Project structure 

Snakemake provides recommendations regarding project structure. 
```
├── .gitignore
├── README.md
├── LICENSE.md
├── workflow
│   ├── rules
|   │   ├── module1.smk
|   │   └── module2.smk
│   ├── envs
|   │   ├── tool1.yaml
|   │   └── tool2.yaml
│   ├── scripts
|   │   ├── script1.py
|   │   └── script2.R
│   ├── notebooks
|   │   ├── notebook1.py.ipynb
|   │   └── notebook2.r.ipynb
│   ├── report
|   │   ├── plot1.rst
|   │   └── plot2.rst
|   └── Snakefile
├── config
│   ├── config.yaml
│   └── some-sheet.tsv
├── results
└── resources
```

Following the [documentation](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html#distribution-and-reproducibility): 
> The workflow code goes into a subdirectory called `workflow`, while the configuration is stored in a subdirectory called `config`. Inside of the `workflow` subdirectory, the central `Snakefile` marks the entrypoint of the workflow (it will be automatically discovered when running snakemake from the root of above structure).

> Workflows that are set up in above structure can be more easily re-used and combined via the [Snakemake module system](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html#use-with-modules). Such deployment can even be automated via [Snakedeploy](https://snakedeploy.readthedocs.io/en/latest/). Moreover, by publishing a workflow on Github and following a set of additional rules the workflow will be automatically included in the [Snakemake workflow catalog](https://snakemake.github.io/snakemake-workflow-catalog/), thereby easing discovery and even automating its usage documentation.


### Containerization

Containers, like Docker, provide a lightweight and portable way to package and run applications. They encapsulate an application and its dependencies, ensuring it runs consistently across different environments. Unlike virtual machines, containers share the host system's kernel but isolate the application's processes, filesystem, and resources. This makes containers more efficient in terms of performance and resource usage, allowing for quick startup times and easy scalability. Docker is a popular containerization platform that simplifies creating, deploying, and managing containers, making it an essential tool for modern software development and deployment.

To use containers with Snakemake, we add the `container` keyword inside a rule, linking either to a local or external container image:

```bash
rule copy:
    container:
        "docker://alpine:3.14"
    input:
	    "hello.txt"
    output: 
	    "hello_copy.txt"
    shell:
	    "cp {input} {output}"
```

To execute that Snakefile, we have to add the `--software-deployment-method` flag:
```bash
snakemake --cores 1 --software-deployment-method apptainer
```
When executed, Snakemake will run the `copy` job inside the `Alpine` (one of the Linux operating systems) image. Note that running that code requires the `apptainer` command to be available on your system.


## Summary

Presented features are only a subset of all things that Snakemake has to offer. Feel free to check the [official documentation](https://snakemake.github.io) for the latest updates! There are also tutorials that cover most of the basics.

Snakemake offers plenty of features that automate the process of creating, running and overseeing the execution. You can document your workflow using DAG and reports. It is easier to make better usage of resources thanks to the implicit parallelization. Error recovery helps with dealing with errors, which are inevitable.

So far, we only used a personal computer. In that mode, Snakemake is using a so-called `local executor`, which uses resources of our computer. After a short break, you will learn how to use other types of executors, that allow for integrating with supercomputers and leveraging resources that they offer.
