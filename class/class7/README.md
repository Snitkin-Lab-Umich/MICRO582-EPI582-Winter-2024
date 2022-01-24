Class 7 – Resistome analysis
===========================

Goal
----

In class6, we learned how to perform basic and functional genome annotation using Prokka and Eggnog. Now we will up the ante and do some more sophisticated comparative genomics analyses!

- We will use the tool [ARIBA - Antimicrobial Resistance Identification By Assembly](https://github.com/sanger-pathogens/ariba/wiki) to identify the complete antibiotic resistome in our genomes.
- Explore Ariba summary reports to gain insights into the types of resistance genes that our genome contains. 
- Then, we will move beyond antibiotic resistance, and look at the entire pan genome structure of our dataset.

![roadmap](comp_genomics.png)


For our analysis today, We will be looking at 8 *Klebsiella pneumoniae* genomes from human and environmental sources. Six of these genomes are from [this paper](https://www.pnas.org/content/112/27/E3574), and the other two are sequences from our lab. We are interested in learning more about potential differences in the resistomes of human and environmental isolates. 


Identify antibiotic resistance genes with [ARIBA](https://github.com/sanger-pathogens/ariba) directly from paired end reads
----------------------------------------------------------

ARIBA identifies antibiotic resistance genes by running local assemblies and can also be used for MLST calling. The input is a FASTA file of reference sequences (antibiotic resistance genes) and paired sequencing reads. ARIBA reports which of the reference sequences were found, plus detailed information on the quality of the assemblies and any variants between the sequencing reads and the reference sequences.


[ARIBA](https://github.com/sanger-pathogens/ariba/wiki) (Antimicrobial Resistance Identification By Assembly) is a tool that identifies antibiotic resistance genes by running local assemblies. The way it works is - takes the input paired end reads and maps them to the reference sequences contained in the provided database. If the reads map with sufficient coverage, it then assembles them into contigs and reports presence/absence of an antibiotic reference sequence.

The input is a FASTA file of reference sequences (this can be a custome database of sequences or a public database containing all the preidentified anitibiotic resistance genes) and paired sequencing reads. 
ARIBA reports which of the reference sequences were found, plus detailed information on the quality of the assemblies and any variants between the sequencing reads and the reference sequences.

ARIBA is compatible with various databases and also contains a utility to download these databases - argannot, card, megares, plasmidfinder, resfinder, srst2_argannot, vfdb_core. 

Today, we will be working with the [card](https://card.mcmaster.ca/) database (`ariba/data/CARD/` in your `class7` directory).

Now let's look at the full spectrum of antibiotic resistance genes in our *Klebsiella* genomes!

> ***i. Run ARIBA on input paired-end fastq reads for resistance gene identification.***

The fastq reads are in the `ariba/data/kpneumo_fastq/` directory. 

```
# navigate to ariba directory
cd /scratch/micro612w21_class_root/micro612w21_class/username/day2pm

# or

d2a

cd ariba

# load conda environment if not already loaded
conda activate day2pm

# look at ariba commands
less ariba.sbat
```

Explore ARIBA summary reports
-----------------------------

ARIBA has a summary function that summarises the results from one or more sample runs of ARIBA and generates an output report with various level of information determined by the `-preset` parameter. The parameter `-preset minimal` will generate a minimal report showing only the presence/absence of resistance genes whereas `-preset all` will output all the extra information related to each database hit such as reads and reference sequence coverage, variants and their associated annotations (if the variant confers resistance to an antibiotic) etc.

```
# look at ariba output
ls results
ls results/card
ls results/card/*

# make directory for ariba results

ariba summary --preset minimal results/kpneumo_card_minimal_results results/card/*/report.tsv

ariba summary --preset all results/kpneumo_card_all_results results/card/*/report.tsv
```

The ARIBA summary generates three output:

1. `kpneumo_card*.csv` file that can be viewed in your favorite spreadsheet program (e.x. Microsoft Excel).

2. `kpneumo_card*.phandango.{csv,tre}` that allow you to view the results in [Phandango](http://jameshadfield.github.io/phandango/#/). You can drag-and-drop these files straight into Phandango.

Lets copy these  files, along with a metadata file, to the local system using cyberduck or scp.

```
mkdir ~/Desktop/micro612
mkdir ~/Desktop/micro612/day2pm

scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day2pm/ariba/results/kpneumo_card* ~/Desktop/micro612/day2pm
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day2pm/ariba/data/kpneumo_source.tsv ~/Desktop/micro612/day2pm
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day2pm/ariba/data/mlst_typing/kpneumo_mlst.tsv ~/Desktop/micro612/day2pm

```

Drag and drop these two files onto the [Phandango](http://jameshadfield.github.io/phandango/#/) website. What types of resistance genes do you see in these *Klebsiella* genomes? 

Now, fire up RStudio and read in the ARIBA full report `kpneumo_ariba_all_results.csv` so we can take a look at the results.

```
# Read in data
ariba_full  = read.csv(file = '~/Desktop/micro612/day2pm/kpneumo_card_all_results.csv', row.names = 1)
rownames(ariba_full) = gsub('_1|_R1|/report.tsv|card/|results/','',rownames(ariba_full))

# Subset to get description for each gene
ariba_full_match = ariba_full[, grep('match',colnames(ariba_full))]

# Make binary for plotting purposes
ariba_full_match[,] = as.numeric(ariba_full_match != 'no')

# Make a heatmap!

# install pheatmap if you don't already have it installed 
#install.packages('pheatmap')

# load pheatmap
library(pheatmap)

# load metadata about sample source
annots = read.table('~/Desktop/micro612/day2pm/kpneumo_source.tsv',row.names=1)
colnames(annots) = 'Source'

# plot heatmap
pheatmap(ariba_full_match,annotation_row = annots)
```

Bacteria of the same species can be classified into different sequence types (STs) based on the sequence identity of certain housekeeping genes using a technique called [multilocus sequence typing (MLST)](https://en.wikipedia.org/wiki/Multilocus_sequence_typing). The different combination of these house keeping sequences present within a bacterial species are assigned as distinct alleles and, for each isolate, the alleles at each of the seven genes define the allelic profile or sequence type (ST). Sometimes, different sequence types are associated with different environments or different antibiotic resistance genes. We want to know what sequence type(s) our genomes come from, and if there are certain ones that are associated with certain sources or certain antibiotic resistance genes. 

We already pre-ran Ariba MLST on all 8 of our *K. pneumonia* genomes. Use the MLST results kpneumo_mlst.tsv that we previously downloaded to add a second annotation column to the heatmap we created above to visualize the results. 

Go to your R studio and overlay MLST metadata as an additional row annotation to your previous heatmap

```
#read in MLST data
annots_mlst = read.table('~/Desktop/micro612/day2pm/kpneumo_mlst.tsv',row.names=1)

#make sure order of genomes is the same as source annotation
annots_mlst$ST = annots_mlst[row.names(annots),]

#paste together annotations
Row_annotations <- cbind(annots, annots_mlst) 

#change from numeric to character, so that heatmap doesn't treat ST as continuous variable
Row_annotations$ST = as.character(Row_annotations$ST)

# Assign colors to Sequence Types
annoCol <- list(ST=c("11"="blue", "221"="red", "230"="orange", "258"="grey"))

#create new heatmap with source and mlst
pheatmap(ariba_full_match,annotation_row = Row_annotations, annotation_colors = annoCol)
```


The commands and steps that were used for MLST typing are given below:

<details>
  <summary>Solution</summary>

**Dont run this exercise**

Steps:
1. Check if you have an MLST database for your species of interest using `ariba pubmlstspecies`.
2. Download your species MLST database. You can look at the manual or run the command `ariba pubmlstget -h` to help figure out how to download the correct MLST database. I would suggest downloading it to the `data` directory. 
3. Copy the `ariba.sbatch` file to a new file called `mlst.sbatch`.
4. Modify the `mlst.sbatch` script in the following ways:
    1. Change the database directory to the _K. pneumoniae_ MLST database you just downloaded.
    1. Change the `mkdir` line to make a `results/mlst` directory.
    1. Modify the output directory `outdir` line: Change `card` to `mlst`.
5. Submit the `mlst.sbatch` script. It should take about 7 minutes to run.
6. Once the run completes, run `scripts/summarize_mlst.sh results/mlst` to look at the MLST results. If you want, you can save it to its own file. What sequence types are present? 


```
# Make sure you are in ariba directory under day2pm folder and running the below commands from ariba directory.
d2a
cd ariba

# Check if you have an mlst database for your species of interest
ariba pubmlstspecies

# Download your species mlst database
ariba pubmlstget "Klebsiella pneumoniae" data/MLST_db

# Set ARIBA database directory to the get_mlst database that we just downloaded.
db_dir=data/MLST_db/ref_db/

# Run ariba mlst with this database
samples=$(ls data/kpneumo_fastq/*1.fastq.gz) #forward reads

# Generate mlst folder under results folder to save ariba MLST results
mkdir results/mlst_typing

# Run for loop, where it generates ARIBA command for each of the forward end files.
for samp in $samples; do   
samp2=${samp//1.fastq/2.fastq} #reverse reads   
outdir=results/mlst_typing /$(echo $samp | cut -d/ -f3 | sed 's/_.*1.fastq.gz//g')
echo "Results will be saved in $outdir"
echo "Running: ariba run --force $db_dir $samp $samp2 $outdir  #ariba command "
ariba run --force $db_dir $samp $samp2 $outdir  #ariba command 
done

# Once the run completes, run summarize_mlst.sh script to print out mlst reports that are generated in current directory
bash scripts/summarize_mlst.sh results/mlst

```
</details>


