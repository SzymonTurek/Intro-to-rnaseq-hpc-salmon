---
title: Data Management and Project Organization
author: Mary Piper, Meeta Mistry, Michael Steinbaugh, Radhika Khetani
date: Wednesday, September 20, 2017
duration: 35
---

[Data management slides]: https://github.com/hbc/NGS-Data-Analysis-long-course/raw/Fall_2016/sessionI/slides/data_management.pdf
[SRA]: http://www.ncbi.nlm.nih.gov/sra  "Sequence Read Archive"

## Learning Objectives

- Recognize the need for data management.
- Plan a good genomics experiment and getting started with project organization.
- Explain the RNA-seq experiment and its objectives.
- Define "metadata" and describe it for the example experiment.

## Data Management

One of the most important parts of research that involves large amounts of data, is how best to manage it. We tend to prioritize the analysis, but there are many other important aspects that are  often overlooked in the excitement to get a first look at new data. 

The data management lifecycle displayed below, courtesy of the [HMS Data Management Working Group](https://datamanagement.hms.harvard.edu/hms-data-management-working-group), illustrates some things to consider beyond the data creation and analysis components:

<img src="../img/data_life_cycle_gouldv2.png" width="350">

_Image aquired from the [Harvard Biomedical Data Management Website](https://datamanagement.hms.harvard.edu/hms-data-lifecycle)_

We will cover some parts of this lifecycle by talking about best practices for the **Research** half of the above lifecycle. Later in this workshop we will talk a little more about the data storage. For more information about the full lifecycle and more guidelines for data management, please look at the resources linked below.

**Resources**

* The [HMS Data Management Working Group (DMWG)'s website](https://datamanagement.hms.harvard.edu/)
* A guide from the [Harvard library](http://guides.library.harvard.edu/dmp).
* Sign-up for the [DMWG quarterly newsletter](https://datamanagement.hms.harvard.edu/dmwg-newsletter) for helpful tips, classes and events related to data management

### Planning

You should approach your sequencing project in a very similar way to how you do a biological experiment, and ideally, begins with **experimental design**. We're going to assume that you've already designed a beautiful sequencing experiment to address your biological question, collected appropriate samples, and that you have enough statistical power.

During this stage it is important to keep track of how the experiment was performed and clearly tracking the source of starting materials and kits used. It is also best practice to include information about any small variations within the experiment or variation relative to standard experiments. 

### Organization

Every computational analysis you do is going to spawn many files, and inevitability you'll want to run some of those analyses again. For each experiment you work on and analyze data for, it is considered best practice to get organized by creating a planned storage space (directory structure).

We will start by creating a directory that we can use for the rest of the RNA-seq session. Log into O2 using `ssh` if you are not already in and start an interactive session on the cluster:

```bash
$ srun --pty -p interactive -t 0-12:00 --mem 1G --reservation=HBC1 /bin/bash
```

Next, let's make sure that you are in your home directory.

```bash
$ pwd
```

This should return `/home/username`.
`

Before we start any analysis, we will create a project directory and set up the following structure to keep files organized. 

```bash
rnaseq
  ├── logs
  ├── meta
  ├── raw_data  
  ├── results
  └── scripts
```

*This is a generic structure and can be tweaked based on personal preference and analysis workflow.*

- `logs`: to keep track of the commands run and the specific parameters used, but also to have a record of any standard output that is generated while running the command. 
- `meta`: for any information that describes the samples you are using, which we refer to as [metadata](https://datamanagement.hms.harvard.edu/metadata-overview). We will discuss this in more detail as it pertains to our example dataset, later in this lesson.
- `raw_data`: for any **unmodified** (raw) data obtained prior to computational analysis here, e.g. FASTQ files from the sequencing center. We strongly recommend leaving this directory unmodified through the analysis.
- `results`: for output from the different tools you implement in your workflow. Create sub-folders specific to each tool/step of the workflow within this folder. 
- `scripts`: for scripts that you write and use to run analyses/workflow.

Here, you can use the parents flag (`-p` or `--parents`) with `mkdir` to complete the file path by creating any parent directories that do not exist. In our case, we have not yet created the `rnaseq` directory and so since it does not exist it will be created. This flag can be very useful when scripting workflows. 


```bash

$ mkdir -p rnaseq/logs rnaseq/meta rnaseq/raw_data rnaseq/results rnaseq/scripts
``` 

Verify that the project directory and subdirectories now exist.

```bash

$ cd rnaseq
$ ls -l

```

Let's populate the `rnaseq/` project with our example RNA-seq FASTQ data.

The FASTQ files are located inside `~/unix_lesson/raw_fastq/`, and we need to copy them to `raw_data/`. We can match them by file extension with `*.fq`.

```bash
$ cp /n/groups/hbctraining/unix_lesson/raw_fastq/*.fq ~/rnaseq/raw_data/
```

> **NOTE**: When obtaining data from your sequencing facility, the data will not be stored on O2 and so a simple copy command (`cp`) will not suffice. The raw sequence data will likely be located on another remote computer/server that is hosted by the sequencing facility and you will be given login credentials to access it. To copy it over you can use commands like `rsync`, `wget` or `scp`. These are all commands that can help securely copy the data over to the appropriate location on O2. We have some information [linked here](https://hbctraining.github.io/In-depth-NGS-Data-Analysis-Course/sessionVI/lessons/more_bash.html#copying-files-to-and-from-the-cluster-) if you would like to learn more. 

Perfect, now the structure of `rnaseq/` should look like this:

```bash
rnaseq
  ├── logs
  ├── meta
  ├── raw_data
  │   ├── Irrel_kd_1.subset.fq
  │   ├── Irrel_kd_2.subset.fq
  │   ├── Irrel_kd_3.subset.fq
  │   ├── Mov10_oe_1.subset.fq
  │   ├── Mov10_oe_2.subset.fq
  │   └── Mov10_oe_3.subset.fq
  ├── results
  └── scripts
```

> #### File naming conventions
> 
> Another aspect of staying organized is making sure that all the filenames in an analysis are as consistent as possible, and are not things like `alignment1.bam`, but more like `20170823_kd_rep1_gmap-1.4.bam`. [This link](https://datamanagement.hms.harvard.edu/file-naming-conventions) and [this slideshow](http://www2.stat.duke.edu/~rcs46/lectures_2015/01-markdown-git/slides/naming-slides/naming-slides.pdf) have some good guidelines for file naming dos and don'ts.


### Documentation

**Documentation doesn't stop at the sequencer!** Keeping notes on what happened in what order, and what was done, is essential for reproducible research.

#### Log files

In your lab notebook, you likely keep track of the different reagents and kits used for a specific protocol. Similarly, recording information about the tools and parameters is important for documenting your computational experiments. 

- **Make note of the software you use.** Do your research and find out what tools are best for the data you are working with. Don't just work with tools that you are able to easily install.
- **Keep track of software versions.** Keep up with the literature and make sure you are using the most up-to-date versions.
- **Record information on parameters used and summary statistics** at every step (e.g., how many adapters were removed, how many reads did not align)
    - A general rule of thumb is to test on a single sample or a subset of the data before running your entire dataset through. This will allow you to debug quicker and give you a chance to also get a feel for the tool and the different parameters.
    - Different tools have different ways of reporting log messages and you might have to experiment a bit to figure out what output to capture. You can redirect standard output with the `>` symbol which is equivalent to `1> (standard out)`; other tools might require you to use `2>` to re-direct the `standard error` instead.
 
#### README files

After setting up the directory structure and when the analysis is running it is useful to have a **[README file](https://datamanagement.hms.harvard.edu/readme-files) within your project directory**. This file will usually contain a quick one line summary about the project and any other lines that follow will describe the files/directories found within it. An example README is shown below. Within each sub-directory you can also include README files to describe the analysis and the files that were generated.

```
## README ##
## This directory contains data generated during the Intro to RNA-seq course
## Date: 

There are six subdirectories in this directory:

raw_data : contains raw data
meta:  contains...
logs:
results:
scripts:
```

*** 

### Homework exercise

- Take a moment to create a README for the `rnaseq/` folder (hint: use `vim` to create the file). Give a short description of the project and brief descriptions of the types of files you will be storing within each of the sub-directories. 

***

## Exploring the example dataset

The dataset we are using is part of a larger study described in [Kenny PJ et al., *Cell Rep* 2014](http://www.ncbi.nlm.nih.gov/pubmed/25464849). The authors are investigating interactions between various genes involved in Fragile X syndrome, a disease of aberrant protein production, which results in cognitive impairment and autistic-like features. **The authors sought to show that RNA helicase MOV10 regulates the translation of RNAs involved in Fragile X syndrome.**

### Raw data

From this study we are using the [RNA-seq](http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE50499) data which is publicly available in the [Sequence Read Archive (SRA)](https://www.ncbi.nlm.nih.gov/sra/?term=SRP029367).

> **NOTE:** If you are interested in how to obtain publicly available sequence data from the SRA we have some materials on this [linked here](https://hbctraining.github.io/Accessing_public_genomic_data/lessons/downloading_from_SRA.html).

### Metadata

In addition to the raw sequence data we also need to collect **information about the data**, also known as **metadata**.  We are usually quick to want to begin analysis of the sequence data (FASTQ files), but how useful is it if we know nothing about the samples that this sequence data originated from? Some relevant metadata for our dataset is provided below:

* The RNA was extracted from **HEK293F cells** that were transfected with a **MOV10 transgene**, **MOV10 siRNA**, or an **irrelevant siRNA**.  (*For this workshop we won't be using the MOV10 knock down samples.*)
* The libraries for this dataset are **stranded** and were generated using the standard Tru-seq prep kit (using the dUTP method). 
* Sequencing was carried out on the **Illumina HiSeq-2500** and **100bp single end** reads were generated. 
* The full dataset was sequenced to **~40 million reads** per sample, but for this workshop we will be looking at a small subset on chr1 (~300,000 reads/sample).
* For each group we have three replicates as described in the figure below.

![Automation](../img/exp_design.png)

## Analysis Workflow

We will be using the dataset described above to perform and end-to-end differential expression analysis. We will begin with the raw sequence reads and go step-by-step through the workflow depicted below, describing in detail some theory, the tools/algorithms and how they work, and the different file formats encountered along the way.

<img src="../img/full_workflow_2019.png">



---

*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

* *The materials used in this lesson were derived from work that is Copyright © Data Carpentry (http://datacarpentry.org/). 
All Data Carpentry instructional material is made available under the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0).*
* *Adapted from the lesson by Tracy Teal. Original contributors: Paul Wilson, Milad Fatenejad, Sasha Wood and Radhika Khetani for Software Carpentry (http://software-carpentry.org/)*

