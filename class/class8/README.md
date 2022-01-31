Class 8 – Data download and genome comparison
=============================================

Goal
----

- In this class, we will learn about NCBI public database and how to download entire datasets pertianing to a research study using SRAtoolkit.
- We will then apply the QC skills that we learned in our previous class such as FastQC and MultiQC to assess the downloaded dataset. (Need to decide if we are going this route)
- We will then do a quick genomic comparison of the downloaded dataset using Mashtree. 

### Databases hosted by NCBI

![SRA](anatomy_of_SRA_submission.png)

The National Center for Biotechnology Information (NCBI) provides bioinformatics tools and hosts approximately 40 online literature and molecular biology databases. Some of the most used and popular databases are PubMed, Genbank, SRA etc. MOst of the NCBI databases are linked together through unique accession numbers and therfore provide a comprehensive resource for biomedical research. For this class, we will focusing on SRA - Sequence read archive database which is the largest publicly available repository of high throughput raw sequencing and alignment data. 

SRA can be searched independently, or SRA records associated with a specific BioProject or BioSample accessions are linked from their respective records.

Each SRA record is given a unique accession number based on the source database (SRA, European Bioinformatics Institute (EBI), or DNA Data Bank of Japan (DDBJ)), and the type of record (Study, Sample, Experiment, Run):

- Study (e.g., the SRA record associated with a specific BioProject): SRP#, ERP#, or DRP#
- Sample (e.g.,the SRA record associated with a specific BioSample): SRS#, ERS#, or DRS#
- Experiment (e.g., the SRA record for a specific experiment or run(s)): SRX#, ERX#, or DRX#
- Run (e.g., the SRA record for a specific run): SRR#, ERR#, or DRR#

A BioProject is a collection of biological data related to a large-scale research effort such as genome sequencing, epigenomic analyses, genome-wide association studies (GWAS) and variation analyses. it provides a single place to find links to the diverse data types generated for that research project. Whereas, The BioSample database contains descriptions of biological source materials used in experimental assays. Each Biosample under a Bioproject can be accessed through a SRA Run accession number that lets you download the sequencing data as well as the metadata associated with it.

NCBI provides a suite of command line tools called Entrez Direct also known as E-utilities to access the metadata stored in its various databases. The three main tools of E-utilities are - esearch, esummary and xtract that lets you query any of the NCBI databases and extract the metadata associated with the query. The query can be anything(Bioproject, Biosample, SRA accession, Genbank assembly accession). 

Here is the command that we used to extract metadata information for the research study which performed a detailed genomic analysis of K. pneumoniae's diversity, population structure, virulence, and antimicrobial resistance around the world. 

```
conda activate class8_sratools

esearch -h

esearch -db sra -query PRJEB2111 | esummary | xtract -pattern DocumentSummary -element Experiment@acc,Run@acc,Platform@instrument_model,Sample@acc > PRJEB2111-info.tsv
```

The Bioproject associated with the study is PRJEB2111. Esearch will return a Edirect object for your query that is then summarized by esummary in XML format. Xtract is then used to extract the metadata elements from this XML format.


Download datasets from NCBI using SRA toolkit
------------------------------------------------

We will now use fasterq-dump tool available from SRA toolkit to download sequencing data for each of the SRA runs that we just saved to PRJEB2111-info.tsv file.


```
wd

mkdir class8

cd class8/

conda activate class8_sratools

fasterq-dump -h

# This is how the data was downloaded. We have already downloaded the data in /scratch/epid582w22_class_root/epid582w22_class/shared_data/data/class8
for accession in $(cut -f2 PRJEB2111-info_subset.tsv); do printf "\n  Working on: ${accession}\n\n"; fasterq-dump ${accession};  done

Or

cut -f2 PRJEB2111-info_subset.tsv | parallel fasterq-dump {}

ls
```

Compare genomes using Mashtree
------------------------------

There are various ways to process these sequence data and perform genome comparison. But using Mash distances and generating a tree based on this Mash distances is the fastest way to get a quick and dirty picture of how each of these samples are related to each other. Mash stands for MinhASH which is the name of algorithm that this disctance estimation is based on. In the min-hash algorithm, all kmers are recorded and transformed into integers using hashing and a Bloom filter (Bloom, 1970). These hashed kmers are sorted and only the first several kmers are retained. The kmers that appear at the top of the sorted list are collectively called the sketch. Any two sketches can be compared by counting how many hashed kmers they have in common. Because min-hash creates distances between any two genomes, min-hash values can be used to rapidly cluster genomes into trees using the neighbor-joining algorithm. 


![mash](Mash.png)



```
cp /scratch/epid582w22_class_root/epid582w22_class/shared_data/data/class8/mashtree.sbat ./

conda activate class8_mashtree

nano mashtree.sbat

sbatch mashtree.sbat

```

