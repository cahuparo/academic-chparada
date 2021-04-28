---
date: '2020-01-15'
title: 'Long read assembly via CANU using HPC'
summary: This bioinformatic notes include step by step work towards assembling long reads using canu.
slug: How-to-run-canu-in-Henry2
categories:
  - bioinformatics
  - Genomics
  - HOW2DO
tags:
  - how2do
  - genomics

# toc: true  # Show table of contents? true/false
# type: docs  # Do not modify.
# menu:
#  post:
#    name: canu
#    weight: 1

projects: []

---
------
## What is all this about?

This bioinformatic notes include step by step work towards assembling long reads using [canu](https://canu.readthedocs.io/en/latest/). Notice that this how to do notes are based on Henry2 capabilities. Once your sequencing run is complete, the sequencing lab will most likely provide the following files:

- m54163.scraps.bam.pbi
- m54163.sts.xml
- **m54163.subreads.bam**
- **m54163.subreads.bam.pbi**
- m54163.adapters.fasta                     
- **m54163.subreadset.xml**
- m54163.baz2bam_1.log                      
- m54163.transferdone
- m54163.scraps.bam    

**Bold** files are some of the most important!

##### What are subreads?

Subreads represent the region of DNA that was sequenced, depending of how many times the polymerase read that region you may have several subreads for a single genomic region. 

## Prerequisites and installation in HPC

The following list includes all programs we need to install in the HPC:

- [x] bamtools
- [x] canu

If you need help installing spack. Check out this tutorial. Remmeber you need to have access to `share` and `usrapps` directories. From your HPC instance you can type:

```sh
source /usr/local/usrapps/PIunityID/spack/spack_use.csh #To activate spack

spack list bam #searches for bamtools

spack install bamtools #Installs bamtools

spack install canu #Installs canu


```

## The pipeline using CANU

### 1) Untar sequencing data and convert `subreads.bam` to `subreads.fastq` using `bamtools`

Before unziping sequencing, we need to move the files to the HPC where we will run the pipeline. Use `scp` to move files into HPC. Prepare a job script to acomplish this step with the following task: 

- Load bamtools 
- Unzip files
- Extract all reads from bam file.


```sh
#!/bin/csh

## Untar sequencing data and convert subreads.bam to subreads.fastq using bamtools

#BSUB -o out.%J
#BSUB -e err.%J 
#BSUB -W 72:00
#BSUB -n 16
#BSUB -R "rusage[mem=16000] span[ptile=8]"
#BSUB -x 
#BSUB -J prep_cfim_canu_assembly

source /usr/local/usrapps/PIunityID/spack/spack_use.csh #To activate spack

spack load bamtools

tar -xvf SEQUENCING_files.tar 
cd SEQUENCING_files

#Extract all reads
bamtools convert -format fastq -in your.subreads.bam -out all_your.subreads.fastq
```

### 2) Read correction using  `-correct` 

Canu's read correction task will replace the original noisy read sequences with consensus sequences computed from overlapping reads. This task takes a while. It is best to run it by itself. Prepare the following job script:

```bash
#!/bin/csh

## Read correction using -correct 

#BSUB -o out.%J
#BSUB -e err.%J 
#BSUB -W 96:00
#BSUB -n 16
#BSUB -R "rusage[mem=16000] span[ptile=8]"
#BSUB -x 
#BSUB -J correction

source /usr/local/usrapps/PIunityID/spack/spack_use.csh #To activate spack

spack load canu #To load canu

canu -correct -p assembly_all -d assembly_all_canu_pacbio genomeSize=40m \
	-pacbio-raw /path/to/all.your.subreads.fastq \
	-maxMemory=16 \
	-maxThreads=8 \
	usegrid=0
```

### 3)Read triming using `-trim` 

Canu's read trimming task will use overlapping reads to decide what regions of each read are high-quality sequence, and what regions should be trimmed. After trimming, the single largest high-quality chunk of sequence is retained.

The following job is designed for trimming the **all reads** data set in the HPC:

```bash
#!/bin/csh

## Read triming using -trim

#BSUB -o out.%J
#BSUB -e err.%J 
#BSUB -W 96:00
#BSUB -n 32
#BSUB -R "rusage[mem=32000] span[ptile=16]"
#BSUB -x 
#BSUB -J trim_assembly_all

source /usr/local/usrapps/PIunityID/spack/spack_use.csh #To activate spack

spack load canu

#Read triming for all 
canu -trim -p assembly_all -d assembly_all_canu_pacbio genomeSize=40m \
	-pacbio-corrected /path/to/assembly_all.correctedReads.fasta.gz \
	-maxMemory=32 \
  -maxThreads=16 \
  usegrid=0
        
```

### 4) Assembly with 2.5% error rate

Finally assembly!
 

The HPC job looks like this:

```sh
#!/bin/csh

## Assembly with 2.5% error rate

#BSUB -o out.%J
#BSUB -e err.%J 
#BSUB -W 120:00
#BSUB -n 16
#BSUB -q shared_memory
#BSUB -R "rusage[mem=32000] span[hosts=1]"
#BSUB -x 
#BSUB -J assembly_all_0.025

source /usr/local/usrapps/PIunityID/spack/spack_use.csh #To activate spack

spack load canu

set numCores=`lscpu | grep "^CPU(s)" | awk '{print $2}’`
echo $numCores

#Assembly with 2.5% error rate 
#For all
canu -assemble -p assembly_all -d assembly_all_erate-0.025 \
        genomeSize=40m \
        correctedErrorRate=0.025 \
        -pacbio-corrected /path/to/assembly_all.trimmedReads.fasta.gz \
        -maxMemory=32 \
        -maxThreads=$numCores \
        usegrid=0
        
        
```



### Some useful monitoring commands:

```sh
#Checking if it run:

bjobs -l
tail -f
bhist
 
#To modify time, ##### = jobnumber
bjobs -l #####
bmod -W 96:00 #####

```

This first assembly process produced the following results:

All_0.025

```sh
Total units: 23
Reference: 40000000
BasesInFasta: 31899376
Min: 2043
Max: 5440577
N10: 5440577 COUNT: 1
N25: 4585891 COUNT: 2
N50: 3215283 COUNT: 5
N75: 1194864 COUNT: 10
E-size:2750431.27
```




