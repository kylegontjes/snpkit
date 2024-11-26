[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10](https://img.shields.io/badge/python-3.1-blue.svg)](https://www.python.org/downloads/release/python-310/)

# SNPKIT - Microbial Variant Calling and Diagnostics toolkit.

## Description

SNPKIT is a variant detection workflow that can be easily deployed for infectious disease outbreak investigations and other clinical microbiology projects. The workflow takes Illumina fastq reads and annotated reference genome as input, calls variants using SAMTOOLS, GATK and Freebayes, generates an annotated SNP/Indel matrix for variants diagnostics and visualizations and a phylogenetic tree based on the curated/filtered variants.

## Fork notice
This was forked from Ali's https://github.com/alipirani88/snpkit.git directory for quality improvement. 

## Contents

- [Installation](#installation)
- [Quick start](#quick-start)
- [Input](#input)

## Installation

The pipeline can be set up in two easy steps:

> 1. Clone the github directory onto your system.

```
git clone https://github.com/kylegontjes/snpkit.git

```

> 2. Use snpkit/environment.yml and snpkit/environment_gubbins.yml files to create conda environment.

Create two new environments - snpkit and gubbins
```
conda env create -f snpkit/envs/environment.yml -n snpkit
conda env create -f snpkit/envs/environment_gubbins.yml -n gubbins
```

Check installation

```
conda activate snpkit

python snpkit/snpkit.py -h
```

## Quick Start

Lets say you want to detect variants for more than a few hundred samples against a reference genome KPNIH1 and want the pipeline to run in parallel on HPC cluster. 


- Run the first step of pipeline with option "-steps All" that will call variants for samples placed in test_readsdir against a reference genome KPNIH1

```

python snpkit/snpkit.py \
-type PE \
-readsdir /Path-To-Your/test_readsdir/ \
-outdir /Path/test_output_core/ \
-analysis output_prefix \
-index KPNIH1 \
-steps call \
-cluster cluster \
-scheduler SLURM \
-clean

```

- The above command will run the variant calling part of the pipeline on a set of PE reads residing in test_readsdir. 
- The results will be saved in the output directory test_output_core. 
- The reference genome and its path will be detected from the KPNIH1 settings that is set in config file.


The results of variant calling will be placed in an individual folder generated for each sample in the output directory. A log file for each sample will be generated and can be found in each sample folder inside the output directory. 

- Run the second part of the pipeline to generate SNP and Indel Matrices and various multiple sequence alignments outputs.

```
python snpkit/snpkit.py \
-type PE \
-readsdir /Path-To-Your/test_readsdir/ \
-outdir /Path/test_output_core/ \
-analysis output_prefix \
-index reference.fasta \
-steps parse \
-cluster cluster \
-gubbins yes \
-scheduler SLURM

```

This step will gather all the variant call results of the pipeline, generate SNP-Indel Matrices, qc reports and core/non-core sequence alignments that can be used as an input for phylogenetic analysis suchs as gubbins-iqtree-beast.

## Input

The pipeline requires three main inputs - readsdir, name of the reference genome and path to the config file.

**1. readsdir:** Place your Illumina SE/PE reads in a folder and give path to this folder with -readsdir argument. Apart from the standard Miseq/Hiseq fastq naming convention (R1_001_final.fastq.gz), other acceptable fastq extensions are: 

```

- R1.fastq.gz/_R1.fastq.gz, 
- 1_combine.fastq.gz, 
- 1_sequence.fastq.gz, 
- _forward.fastq.gz, 
- _1.fastq.gz/.1.fastq.gz.

```

**2. config:** A high level easy to write YAML format configuration file that lets you configure your system wide runs and specify analysis parameters, path to the installed tools, data and system wide information.

- This config file will contain High level information such as locations of installed programs like GATK, cores and memory usage for running on HPC compute cluster, path to a reference genome, various parameters used by different tools. These settings will apply across multiple runs and samples. 

- The config file stores data in KEY: VALUE pair. 

- An example [config](https://github.com/alipirani88/snpkit/blob/master/config) file with default parameters is included with the installation folder. You can customize this config file and provide it with the -config argument or edit this config file based on your requirements. 

- Parameters for each of the tools can be customised under the 'tool_parameter' attribute of each tool in config file. 

- If you wish to run pipeline in hpc compute environment such as PBS or SLURM, change the number of nodes/cores memory reuirements based on your needs else the pipeline will run with default settings.


**3. index:** a reference genome index name as specified in a config file. For example; if you have set the reference genome path in config file as shown below, then the required value for command line argument -index would be -index KPNIH1

NOTE: The index_name must correspond to the fasta name  
```
[KPNIH1]
# path to the reference genome fasta file.
Ref_Path: /nfs/turbo/umms-esnitkin/data_sharing/reference/KPNIH1/
# Name of reference genome fasta file.
Ref_Name: KPNIH1.fasta
```

Here, Ref_Name is the reference genome fasta file located in Ref_Path. Similarly, if you want to use a different version of KPNIH1 reference genome, you can create a new section in your config file with a different index name.

```
[KPNIH1_V2024]
# path to the reference genome fasta file.
Ref_Path: /nfs/turbo/umms-esnitkin/data_sharing/reference/KPNIH1_V2024/
# Name of reference genome fasta file.
Ref_Name: KPNIH1_V2024.fasta
```

THe pipeline also requires Phaster results of your reference genome to mask phage region. The pipeline assumes that you have placed the reference genome fasta file `KPNIH1.fasta` in folder `/nfs/turbo/umms-esnitkin/data_sharing/reference/KPNIH1/`, a genbank annotation file with extension `.gbf` and phaster results downloaded from Phaster website for your specific reference genome. The phaster file  that pipeline expects are `summary.txt` and `phage_regions.fna`.

Note: By default, Prokka outputs only .gbk files and not .gbf. You can change the extension of your genbank file to `.gbf` from `.gbk`. This could solve the possible extension requirement issue.

### For detailed information, please refer to the [wiki](https://github.com/alipirani88/snpkit/wiki) page.

