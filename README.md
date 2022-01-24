# MICRO582/EPI582 - Genomic Epidemiology Winter 2022

#### Instructor(s): Evan Snitkin

#### Description: Provide an overview of cutting-edge approaches for microbial genomic analysis, with a focus on their application in the emerging field of genomic epidemiology. Lectures will be reinforced with discussions and extensive hands-on training. Students will gain experience on the command-line and in performing downstream analysis and visualization in R.

#### Date: Winter Term 2022

#### Course Discipline: Epidemiology

### Overview:
------------

### Section - Working with sequence data at the command line
--------------------------------------------------------

#### [Class 1 – Setting up Great Lakes account and Introduction to UNIX](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class1/README.md)
***

- [Sign up for Great Lakes account](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class1/README.md#sign-up-for-great-lakes-account)
- [Intro to Unix](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class1/README.md#intro-to-unix)
- [Navigating directory structure](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class1/README.md#navigating-directory-structure)
- [Data Carpentry: Introducing the Shell & Navigating Files and Directories](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class1/README.md#data-carpentry:-introducing-the-shell-&-navigating-files-and-directories)

#### [Class 2 – Sequence data formats](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class2/README.md)
***

- [Using GREP for pattern matching](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class2/README.md#using-grep-for-pattern-matching)
- [Using for loops to perform same actions on different files](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class2/README.md#using-for-loops-to-perform-same-actions-on-different-files)

#### [Class 3 – Introduction to Great Lakes and HPC](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class3/README.md)
***

- [Working with Files and Directories](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class3/README.md#working-with-files-and-directories)
- [Setting up your compute environment](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class3/README.md#setting-up-your-compute-environment)
- [Loading modules](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class3/README.md#loading-modules)
- [Submit a job to cluster](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class3/README.md#submit-a-job-to-cluster)

#### [Class 4 – Illumina sequencing data and QC](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class4/README.md)
***

- [Quality Control using FastQC](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class4/README.md#quality-control-using-fastqc)
- [Quality Trimming using Trimmomatic](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class4/README.md#quality-trimming-using-trimmomatic)


### Section - Generating sequence data and performing annotation
------------------------------------------------------------

#### [Class 5 – Genome assembly](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class5/README.md)
***

- [Performing assembly on the cluster](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class5/README.md#performing-assembly-on-the-cluster)
- [Assess assembly quality with Quast](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class5/README.md#assess-assembly-quality-with-quast)
- [Aggregate and Assess dataset quality using multiQC](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class5/README.md#aggregate-and-assess-dataset-quality-using-multiqc)

#### [Class 6 – Genome annotation](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class6/README.md)
***

- [Genome annotation using PROKKA](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class6/README.md#genome-annotation-using-prokka)
- [Functional annotation using eggnog](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class6/README.md#functional-annotation-using-eggnog)


#### [Class 7 – Resistome analysis](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class7/README.md)
***

- [Identify antibiotic resistance genes with ARIBA directly from paired end reads](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class7/README.md#identify-antibiotic-resistance-genes-with-ariba-directly-from-paired-end-reads)
- Explore ARIBA summary reports
- Determine AR genes using BLAST

#### [Class 8 – Data download and genome comparison](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class8/README.md)
***

- Download datasets from NCBI using SRA toolkit
- Assess quality of downloaded dataset using fastQC, QUAST, multiQC
- Perform in silico MLST detection using ARIBA
- Perform Genome comparison using 
- Visualize data using iTol

#### [Class 9 – Reference based variant calling](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class10/README.md)
***

- Variant calling using SNIPPY - mapping Resistant sample to susceptible reference genome
- Run Snippy on unknown R to screen for variants in gene of interest
- Explore alignments and variants in IGV


### Section - Basic phylogenetic analysis
-------------------------------------

#### [Class 10 – Intro to RStudio](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class11/README.md)
***

- Introduction to Rstudio


#### [Class 11 – Basic phylogenetic analysis](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class12/README.md)
***

- Overlaying antibiotic resistance on a phylogeny to understand patterns of evolution/spread
- Overlaying geography on a phylogeny to put local isolates in global context
- Overlay origin on a phylogeny to understand how infections with differing epidemiology are related


#### [Class 12 – Genome-wide association studies](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class13/README.md)
***

- Perform bGWAS for antibiotic resistance


#### [Class 13 – Recombination detection](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class14/README.md)
***

- Whole genome alignment with parsnp
- Recombination filtering with Gubbins
- Phylogenetic analysis of outbreak

### Section - Hospital outbreak investigation
----------------------------------------------------

#### [Class 14 – Overview of genomic epidemiology](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class15/README.md)
***




#### [Class 15 – Guest lecture on AMR in public health](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class16/README.md)
***




#### [Class 16 – Transmission in endemic settings](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class17/README.md)
***




#### [Class 17 – Regional outbreaks](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class18/README.md)
***

-  Investigating Regional transmission using Genomic data


#### [Class 18 – Regional endemic spread](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class19/README.md)
***

- Analyze AAC paper in regentrans

#### [Class 19 – Nanopore sequencing](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class20/README.md)
***



#### [Class 20– Guest lecture on AMR surveillance in the environment](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class21/README.md)
***



### Section - Genomic epidemiology of SARS-CoV-2
--------------------------------------------

#### [Class 21 – Guest lecture on COVID genomics in public health](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class22/README.md)
***



#### [Class 22 – SARS-CoV-2 genomics](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class23/README.md)
***

- Examine the impact of data QC on genetic distance distributions
- Identify putative transmission clusters and examine makeup

#### [Class 23 – Dated phylogenetic analysis](https://github.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2022/blob/main/class/class24/README.md)
***

- Perform root-to-tip analysis before and after data QC
- Construct dated tree
- Overlay geographic data on tree

