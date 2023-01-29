Class 7 – Resistome analysis
===========================

Goal
----

In class6, we learned how to perform basic and functional genome annotation using Prokka and Eggnog. Now we will perform some more targeted genome annotation and attempt to identify specific antibiotic resistance genes and mutations in a set of genomes.

- First, we will create custom BLAST databases to identify specific antibiotic resistance genes of interest in a set of genomes. 
- Second, We will use the tool [AMRFinderPlus](https://www.ncbi.nlm.nih.gov/pathogens/antimicrobial-resistance/AMRFinder/) to identify the complete set of antibiotic resistance genes in our genomes (i.e. the resistome) by comparing to a reference database of curated resistance genes and mutations.
- Then, we will write some shell scripts to explore amrfinderplus results 

![roadmap](comp_genomics.png)

For BLAST and AMRFinderPlus, we will be looking at 8 *Klebsiella pneumoniae* genomes from human and environmental sources. Two of the genomes are from [this paper](https://www.pnas.org/content/112/27/E3574), and the other two are sequences from our lab. We are interested in learning more about potential differences in the resistomes of human and environmental isolates. 

Determine which genomes contain KPC genes using [BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi)
----------------------------------------------------
<!---
![blast](comp_genomics_details_blast.png)
--->

Before comparing full resistance gene/mutation content, lets start by looking for the presence of particular genes of interest. Some *K. pneumoniae* harbor a KPC gene that confers resistance to carbapenems, a class of antibiotics of last resort (more information [here](https://www.sciencedirect.com/science/article/pii/S1473309913701907?via%3Dihub) and [here](https://academic.oup.com/jid/article/215/suppl_1/S28/3092084)). 

We will see if any of our samples have a KPC gene, by comparing the genes in our genomes to KPC genes extracted from the antibiotic resistance database ([ARDB](http://ardb.cbcb.umd.edu/)). These extracted genes can be found in the file `blast/data/blast_kleb/ardb_KPC_genes.pfasta`, which we will use to generate a BLAST database.

First, change directories to the working directory and copy class7 directory:

```

wd

cp -r /scratch/epid582w23_class_root/epid582w23_class/shared_data/data/class7 ./ 

cd class7/blast

```

Now activate the class7 conda environment.

```
# If you haven't installed class7 then install it with: conda create -n class7 -c bioconda blast tbb=2020.2 ncbi-amrfinderplus

conda activate class7
```


> ***i. Run makeblastdb on the file of KPC genes to create a BLAST database.***

makeblastdb takes as input: 

1) an input fasta file of protein or nucleotide sequences (`blast_db/ardb_KPC_genes.pfasta`) and 

2) a flag indicating whether to construct a protein or nucleotide database (in this case protein: `-dbtype prot`).

```
makeblastdb -in blast_db/ardb_KPC_genes.pfasta -dbtype prot
```

> ***ii. BLAST K. pneumoniae protein sequences against our custom KPC database.***

Run BLAST! 

The input parameters are: 

1) query sequences - These are protein sequences from all the 4 genomes created by concatenating each Prokka \*.faa output files (`-query kpneumo_four_genomes.faa`), 

2) the database to search against (`-db blast_db/ardb_KPC_genes.pfasta`), 

3) the name of a file to store your results (`-out KPC_blastp_results.tsv`), 

4) output format (`-outfmt 6`), 

5) e-value cutoff (`-evalue 1e-100`), 

6) number of database sequences to return (`-max_target_seqs 1`) (Note that when using large databases, this might not give you the best hit. See [here](https://academic.oup.com/bioinformatics/advance-article/doi/10.1093/bioinformatics/bty833/5106166) for more details.)


```
# Takes around 0m43.271s

blastp -query kpneumo_four_genomes.faa -db blast_db/ardb_KPC_genes.pfasta -out KPC_blastp_results.tsv -outfmt 6 -evalue 1e-100 -max_target_seqs 1
```

Use `less` to look at `KPC_blastp_results.tsv`. Which genomes have a KPC gene?

```
less KPC_blastp_results.tsv
```

[Here](http://www.metagenomics.wiki/tools/blast/blastn-output-format-6) is more information about the content for each of the output file columns.

<!---
- **Exercise:** In this exercise you will try a different type of blasting – blastx. Blastx compares a nucleotide sequence to a protein database by translating the nucleotide sequence in all six frames and running blastp. Your task is to determine which Enterococcus genomes are vancomycin resistant (VRE, vs. VSE) by blasting against a database of van genes. The required files are located in `blast/data/blast_ent` folder in the `day2pm` directory.

Your steps should be:

1) Concatenate the `data/blast_ent/*.fasta` files (VRE/VSE genomes) into a single file (your blast query file) using the `cat` command.
2) Create a blastp database from `data/blast_ent/ardb_van.pfasta`
3) Run blastx
4) Verify that only the VRE genomes hit the database
5) For extra credit, determine which van genes were hit by using grep to search for the hit gene ID in `data/blast_ent/ardb_van.pfasta`

<details>
  <summary>Solution</summary>
  
```
cd blast/data/blast_ent

# Make sure you are in blast_ent folder
cat *.fasta > VRE_VSE_genomes.fasta

makeblastdb -in ardb_van.pfasta -dbtype prot

blastx -query VRE_VSE_genomes.fasta -db ardb_van.pfasta -out van_blastp_results.tsv -outfmt 6 -evalue 1e-100 -max_target_seqs 1

```
</details>

- **Exercise:** Experiment with the `–outfmt` parameter, which controls different output formats that BLAST can produce. You can use `blastp -help | less` to get more information about the different output formats. You can search for the `-outfmt` flag by typing `/outfmt` and then typing `n` to get to the next one.

--->


Identify antibiotic resistance genes with AMRFinderPlus
---------------------------------------------------
[AMRFinderPlus](https://github.com/ncbi/amr/wiki) is a tool made available by NCBI to identify resistance genes and mutations in assembled bacterial genomes. For a detailed description of the AMRFinderTool and how it was built/validated, check out these papers from [2019](https://journals.asm.org/doi/full/10.1128/AAC.00483-19) and [2021](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8208984/). The key to the AMRFinder tool is a highly currated set of antibiotic resistance gene families and resistance conferring mutations. THe initial curation involved  the careful identification of these genes from literature, existing databases and in conjunction with protein-specific experts. Then, a sequence curation was performed to cataolog all the genes/alleles for each resistance gene family, organize them in a hierarchical framework to enable more accurate/meaningful gene/allele assignments and created currated multiple sequence alignments of each gene family. 

When running AMRFinderPlus protein and/or nucleotide sequences are compared to this curated database of antibiotic resistance genes and alleles. There are two methods employed to compare to these databases. The first is using BLAST, with stringent cutoffs to ensure that the hits are specific. The second is the use of a proababilistic model called hidden markov models ([HMMs](https://www.nature.com/articles/nbt1004-1315)). HMMs are commonly used tools in sequence analysis. The input to an HMM is a multiple alignment of the sequences of interest, which is this case are antibiotic resistance proteins in the same family, where families are based on shared ancestry (i.e. homology). Training algorithms are then applied to create a probabilistic model that captures the defining features of the sequences in the alignment (e.g. conserved alanine at position 130, basic amino acid at position 260, etc.). For each protein inputted to AMRFinder, these HMMs are used to scan the protein to detemrine whether its a good match for the protein family. Lastly, AMRFinder employs a decision tree to use BLAST and HMM results to determine which antibiotic resistance genes and alleles are present in an input genome.


For the lab we ran AMRFinderPlus on our four Klebsiella genomes for you, but have provided the sbat scripts that we used. Note the presence of two different sbat scripts amrfinder_commands.sbat and amrfinder.sbat. The amrfinder_commands.sbat script just contains the four calls to AMRFinderPlus, once for each genome. The amrfinder.sbat takes a more robust approach, and examines the directory containing genome assemblies we want to analyze with AMRFinderPlus, creates the appropriate command, and runs it on the cluster! I am hoping to go through that script next class, after we learn some new shell script tricks today!

To run the basic approach, run the following commands:

```
#Test if you can can call amrfinder
amrfinder -h

#Download/Update AMRFINDER database
amrfinder -u

cd amrfinder/

sbatch amrfinder_commands.sbat
```

The results of AMRFinderPlus are in the directory amr_finder_results_all. For each of the four genomes there are two files: i) mutation_report.tsv and ii) .txt. The mutation report provides information on known resistance-conferring protein coding mutations identified in your input genome. The .txt provides a summary of known antibiotic resistance genes detected in your input genome. Today, we will focus our attention on the .txt file.


Let's start by taking a look at the .txt file for one of our genomes using the less command.

```
#Use the -S flag to enable better display of tabular data
less -S ERR025152.txt
```

Even with our -S trick, it's still messy to look at, and hard to see what the headers are. So, let's use our trick from last time to pull the header from our file, and break it apart to see the column numbers as well.

```
head -n 1 ERR025152.txt | tr "\t" "\n" | cat -n
```

To enable us to compare the resistance encoding potential of our environmental and healthcare Klebsiella genomes, we are going to write a handy-dandy shell script that extracts pertinent information from this .txt file. In particular, let's report: i) the number of unique antibiotic resistance classes for which resistance genes are found and ii) the subclass and gene symbol for each amr gene detected.

To do this we are going to edit the shell script amr_finder_res_summary.sh, which takes as a command line argument the name of a .txt file created by AMRFinderPlus. 

```
# Usage
# bash amr_finder_res_summary.sh amr_finder_report
```


So, open it up with nano, and let's start by trying to edit the following part of the code to report the number of unique resistance classes.

```
# Create a Unix command to print out the number of unique subclasses resistance elements are detected for
amr_count=$()

#Report the number of resistance elements
echo "Number of resistance elements detected: $amr_count"
echo
```

To accomplish this goal we are introducing you to a few new concepts. First, we are creating our own custom variable amr_count, which we have done in the context of our .bashrc (i.e. environment variables), but in shell scripts we have only used special variables (e.g. loop variables, command line arguments). To create a new variable you write the name of the variable, and then =. The second new concept we are introducing is using () to assign the output of a unix command to a variable. So, in this example, you will place in the parentheses a unix command to determine the number of unique antibiotic resistance subclasses present in a .txt file provided by AMRFinderPlus.

Hints:
1) You will first want to apply a grep command to select lines that have AMR, as this report also reports on virulence and stress response genes
2) You will want to cut the column corresponding to resistance gene class
3) To get the counts of unique resistance classes you will use sort, uniq and wc

<details>
  <summary>Solution</summary>

# Create a Unix command to print out the number of unique subclasses resistance elements are detected for
amr_count=$(grep AMR $1 | cut -f 12 | sort | uniq | wc -l)
echo "Number of resistance elements detected: $amr_count"
echo

</details>
  
  
Next, let's add in Unix commands to extract two columns from our file - Subclass and Gene Symbol.

```
# Create Unix command to print out the subclass and gene symbol from amrfinder report
# To do this: 1) Select lines that have AMR (all caps)
#             2) Print out the columns Subclass and Gene symbol
#             3) Sort the output by Subclass (hint - using the -k flag you can select number of column to sort by)
#       

echo
```

<details>
  <summary>Solution</summary>

# Create a Unix command to print out the number of unique subclasses resistance elements are detected for
amr_count=$(grep AMR $1 | cut -f 12 | sort | uniq | wc -l)
echo "Number of resistance elements detected: $amr_count"
echo

</details>

<!---


Now that we've finished our shell script, let's apply it to one of our genomes!

```
bash amr_finder_res_summary.sh amr_finder_results_all/ERR025152.txt
```

OK, this is a pretty sweet shell script, but now we want to run it on each of our genomes. We could do this by typing out four commands, but instead, let's write a second shell script that runs our first shell script on all AMRFinderPlus output files in a given directory. This shell script will take a single command line argument (the directory path), and run your shell script on each .txt file in the directory. Here is the usage statement:

```
# Run your amr_finder_res_summary.sh script on all files in an input directory

# Usage
# amr_finder_res_summary_batch.sh amr_finder_results_dir
```

So, fill in the code below the following comment, which provides some hints/instructions:

```
# Write a for loop to run your shell script on each .txt file in the 
# output directory input as the first command line argument (i.e. $1)
# Remember to use echo to print the name of the file so you know which
# output belongs to which file

```

<details>
  <summary>Solution</summary>

for file in $1/*.txt
do
        echo $file      

        echo $prefix
        bash amr_finder_res_summary.sh $file

done

</details>


Finally, let's put it all together and run this shell script to parse AMRFinderPlus results and report on the resistome of each of our four genomes.

```
bash amr_finder_res_summary_batch.sh amr_finder_results_all/
```

What differences do you notice between the antibiotic resistance potential for our two environmental isolates (ERR*) versus our hospital isolates (PCMP*)?



Identify antibiotic resistance genes with [ARIBA](https://github.com/sanger-pathogens/ariba) directly from paired end reads
----------------------------------------------------------

[ARIBA](https://github.com/sanger-pathogens/ariba/wiki) (Antimicrobial Resistance Identification By Assembly) is a tool that identifies antibiotic resistance genes by running local assemblies. The way it works is - takes the input paired end reads and maps them to the reference sequences of antibiotic resistance genes contained in the provided database (e.g. [CARD](https://card.mcmaster.ca/)). If the reads map with sufficient coverage, it then assembles them into contigs and reports presence/absence of an antibiotic reference sequence.

The input is a FASTA file of reference sequences (this can be a custome database of sequences or a public database containing all the preidentified anitibiotic resistance genes) and paired sequencing reads. 
ARIBA reports which of the reference sequences were found, plus detailed information on the quality of the assemblies and any variants between the sequencing reads and the reference sequences.

ARIBA is compatible with various databases and also contains a utility to download these databases - argannot, card, megares, plasmidfinder, resfinder, srst2_argannot, vfdb_core. 

Today, we will be working with the [CARD](https://card.mcmaster.ca/) database (`ariba/data/CARD/` in your `class7` directory).

Now let's look at the full spectrum of antibiotic resistance genes in our *Klebsiella* genomes!

> ***i. Run ARIBA on input paired-end fastq reads for resistance gene identification.***

The fastq reads are in the `ariba/data/kpneumo_fastq/` directory. 

```
# navigate to ariba directory
wd

cd class7/ariba

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
--->


<!---
Lets copy these  files, along with a metadata file, to the local system using cyberduck or scp.

```
mkdir ~/Desktop/epid582w23_class
mkdir ~/Desktop/epid582w23_class/class7

scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/epid582w23_class_root/epid582w23_class/username/class7/ariba/results/kpneumo_card* ~/Desktop/epid582w23_class/class7
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/epid582w23_class_root/epid582w23_class/username/class7/ariba/data/kpneumo_source.tsv ~/Desktop/epid582w23_class/class7
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/epid582w23_class_root/epid582w23_class/username/class7/ariba/data/mlst_typing/kpneumo_mlst.tsv ~/Desktop/epid582w23_class/class7
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
annots_mlst = read.table('~/Desktop/epid582w23_class/class7/kpneumo_mlst.tsv',row.names=1)

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

--->

