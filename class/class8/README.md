Class 8 – Data download and genome comparison
=============================================

Goal
----

- In this class, we will learn about NCBI public database and how to download entire datasets peratianing to a research study using SRAtoolkit.
- We will then apply the QC skills that we learned in our previous class such as FastQC and MultiQC to assess the downloaded dataset.
- We will then do a quick genomic comparison of the downloaded dataset using Mashtree. 

### Add NCBI database stucture descriptions.
### Difference between Bioproject, Biosample, SRA accession.

![SRA](anatomy_of_SRA_submission.png)

Download datasets from NCBI using SRA toolkit
------------------------------------------------

```
islurm

wd

mkdir class8
cd class8/

conda activate class8

fasterq-dump -h

# Use this or just provide them with the SRA accessions. Whichever makes it less confusing. 
# Esearch/Esummary/Xtract can be a little intimidating at first.
# Lets keep it simple and just provide them with the 3o SRA accession from PRJEB2111 stucy.
# esearch -db sra -query PRJEB2111 | esummary | xtract -pattern DocumentSummary -element Experiment@acc,Run@acc,Platform@instrument_model,Sample@acc | head -n30 > PRJEB2111-info_30_samples.tsv

for accession in $(cut -f2 PRJEB2111-info_30_samples.tsv); do printf "\n  Working on: ${accession}\n\n"; fasterq-dump ${accession};  done

ls
```

Compare genomes using Mashtree
------------------------------

```
islurm

time mashtree --numcpus 8 *.fastq --outtree mashtree_faster.dnd --outmatrix mashtree_faster.tsv

#real	6m49.139s
#user	6m32.699s
#sys	0m12.186s

time mashtree --numcpus 8 *.fastq --outtree mashtree_accurate.dnd --outmatrix mashtree_accurate.tsv --mindepth 0

time mashtree_bootstrap.pl --reps 100 --numcpus 8 --min-depth 0 --outtree mashtree_bootstrap.dnd --outmatrix mashtree_bootstrap.tsv --mindepth 0 *.fastq
```
