Class 6 – Genome annotation
===========================

Goal
----
- We will annotate an assembled multidrug resistant Klebsiella pneumoniae genome with PROKKA and learn about different annotation formats
- We will generate richer annotations for our multidrug resistant Klebsiella and an environmental strain using eggnog
- We will use COG ontology to compare the functional categories encoded by our two Klebsiella strains
- We will briefly examine the output of panaroo, which tells us which genes are shared by sets of genomes

Genome annotation using PROKKA
------------------------------

Our genome asembly consists of long nucleotide sequences called contigs. However, having ~5Mb of sequence isn't super useful on it's own! What makes a genome informative are annotations, such that we can gain insight into the functions encoded in a genome and interpret the impact of genetic variation among a set of genomes. Over the past ~20 years there has been a great deal of bioinformatics research into how to identify functional elements in genomes, and in turn how to generate hypotheses regarding the molecular functions they perform and the biological processes in which they participate. Nowadays, there are pipelines available that combine different sets of tools in order to produce a fully annotated genome. For bacteria, the most commonly used tool is called Prokka. Prokka takes as input a genome assembly and performs the following functions:

1. Applies a tool called [Prodigal](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-11-119) to identify putative protein coding sequences.
2. Applies a tool called [RNAmmer](https://academic.oup.com/nar/article-abstract/35/9/3100/2401119) to identify rRNA encoding genes
3. Applies a tool called [Aragorn](https://academic.oup.com/nar/article-abstract/32/1/11/1194008) to identify tRNA encoding genes
4. Applies a tool called [Infernal](https://academic.oup.com/bioinformatics/article/29/22/2933/316439?login=true) to identify small RNA encoding genes

After applying these tools to identify functional elements, it then compares them to databases  of curated sequences (e.g. using BLAST) and transfers annotation. It is important to note that these annotations should be considered as bioinformatically generated hypotheses, with the confidence being associated with how closely related the sequence in the database is to the sequence you are annotating. 

Finally, the results of all of these annotations are stored in a series of standard genome annotation files, including:
1. .ffn - A fasta formated file of the nucleotide sequences of annotated features
2. .faa - A fasta formated file of the amino acid sequences of annotated coding regions
3. .gff - A gff formatted file of genome annotations
4. .gbk - A genbank formatted file of genome annotations and sequences
5. .tsv - A tab-separated value formatted file of genome annotations

With that background, let's try it on for ourselves! We are going to run Prokka on the *Staphylococcus aureus* genome that we assembled last class. Since Prokka annotation is a time intensive run, we will submit an annotation job and go over the results later at the end of this session. 

Lets add Prokka to our environment and check if we can invoke it.

Add this line at the end of your .bashrc file and source it.

```
export PATH=$PATH:/scratch/epid582w23_class_root/epid582w23_class/shared_data/bin/prokka/bin/
export EGGNOG_DATA_DIR=/scratch/epid582w23_class_root/epid582w23_class/shared_data/database/eggnog
```

Let load some perl-modules that prokka requires and invoke prokka's help menu.

```
module load Bioinformatics
module load perl-modules/1.0
module load bioperl/1.7.2
prokka -h
```


Lets copy over class6 data to your class working directory.

```
wd

cp -r /scratch/epid582w23_class_root/epid582w23_class/shared_data/data/class6 ./

cd class6
```

Now edit your email in the slurm script - annotate.sbat and submit the job with sbatch.

It should take around 3 minutes to finish the prokka run.

Next, let's explore some of the output files using our ever growing bag of Unix tricks :). First, lets take a look at the overall summary file that Prokka creates which indicates how many of each type of genomic feature was identified:

```
cd PCMP_H183

less PCMP_H183.txt
```

You'll notice that this file is actually short, so instead of viewing in less it might be easier to dump the contents to the screen. You can do this with the 'cat' command:

```
cat PCMP_H183.txt
```

Next, as an exercise, see if you can write a for loop that goes through the gff file created by Prokka and counts the number of occurences of each type of feature.

<details>
  <summary>Solution</summary>

```
for feat in CDS repeat_region rRNA tmRNA tRNA; 
do 
  #PRINT OUT THE NAME OF THE FEATURE
  echo $feat;
  
  #COUNT THE NUMBER OF OCCURENCES OF THE FEATURE IN THE THIRD COLUMN OF THE gff
  cut -f 3 PCMP_H183.gff| grep $feat | wc -l; 

done

```
</details>

One more quick exercise - apply a Unix command to the appropriate fasta file to count the number of coding sequences and verify that it matches up with your loop.

<details>
  <summary>Solution</summary>

```
  
grep ">" PCMP_H183.faa | wc -l
  
```
</details>



Functional annotation using eggnog
----------------------------------

Prokka provides you with some basic annotations (e.g. putative function of protein coding genes). However, it is often valuable to have richer annotations that place genes into different functional categories and/or pathway assignments such that:

1. You can gain a quick overview of how a genome is partitioned into different functional classes
2. You can compare genomes holistically to get a sense of how they are functionally different (e.g. nutrient sources they can utilize/environments they can grow in)
3. When evaluating the differences between pairs of genomes or sets of genomes (e.g. differences in gene content, gene expression, etc.), you can get a quick sense of the overall functional differences, which can lead to more focused and data-driven hypotheses

To get richer annotation of our genomes we will apply a tool called Eggnog mapper. Eggnog mapper leverages a database of curated sequences (the Eggnog database), and applies an algorithm to map inputted genes to the most likely evolutionary counterpart (i.e. it's [ortholog](https://www.nature.com/articles/nrg3456)). There is a wide array of annotations that are provided, but three of special importance are [Gene Ontology](https://www.nature.com/articles/nrg3456) (GO), [KEGG](https://www.genome.jp/kegg/) and [Clusters of Orthologous Genes](https://www.ncbi.nlm.nih.gov/research/cog-project/)(COG)annotation. GO is an ontology scheme that assigned each gene to a biological process, molecular function and cellular component. One powerful aspect of GO is that it is hierarchical, such that you can categorize the role of genes at different levels (e.g. metabolism -> central carbon metabolism -> glycolysis). KEGG has a multitude of useful databases, but the most commonly used are it's pathway maps, which group genes into biochemical pathways and signalling cascades. 

The primary output file created by Eggnog mapper is a tab-delimited file with an extension `.emapper.annotations` providing different annotations for each gene. If you examine the file using less you will notice that the first three lines are comments, and the fourth line is a set of column headers. In order to explore specific annotations with Unix commands it would be super useful to know which annotations are on which columns. 

To do this you could tediously count across, but you should know by now that we don't like to do things manually :). So, let's check out a cool Unix trick to get what we want! 

There are two new concepts here. 
- The first is the use of the 'tr' command, which we will use to "traslate" the tabs in the file to carriage returns. 
- The second is the use of 'cat' with the -n flag, which provides line numbers to the output. 
- To pull out the line of interest there are a variety of Unix commands we could employ, but to avoid introducing too many new concepts, let's just go with a grep solution:

```
grep "^#query" PCMP_H183_eggnog.emapper.annotations | tr "\t" "\n" | cat -n
```

Next, let's get a sense for how the multidrug resistant Klebsiella genome is distributed among the different COG functional categories. From the above command you can see that the 7th column is COG functional categories. If you check out the file we provide 'COG_functional_categories.txt', you can see that each functional category is represented by a single letter code, and you can also see what the different categories are. Let's now pipe together some Unix commands we have used before to see all the unique COG categories in the Eggnog annotation file. We are also going to use one new flag (-c with uniq), which will tell us how many times each category occurs.

```
grep -v "#" PCMP_H183_eggnog.emapper.annotations | cut -f 7 | sort | uniq -c 
```

From the above you can see that things are somewhat complciated. Different COG categories occur different numbers of times, and some genes are assigned multiple COG categories. Moreover, unless you are a frequent COG user, the single letter codes don't mean much in themselves. To get around these issues, and enable easy comparison of the COG distribution in our multidrug resistant Klebsiella genome, and an environmental Klebsiella genome, we are going to write a shell script.

In our shell script we are going to use the file 'COG_functional_categories.txt', which is a lookup of what each COG single letter stands for. What we want to then do is loop through each COG category from this file, print out the name in the lookup file and then count the number of times it occurs in the Eggnog output. How to do something so fancy though? Well, you already have all the tools to accomplish this! To make it more exciting and useful we are going to put these commands into a shell script that we could then use with any Eggnog mapper output file. 

We have started the code, but you have to add two commands in the for loop to make it work :). Open 'COG_code_counter.sh' in nano and add the appropriate commands below the comments in the for loop.

```
#GO THROUGH EACH COG 1-LETTER CODE AND PRINT OUT NAME AND COUNT FROM EGG NOG OUTPUT FILE
for COG_code in `cut -f 1 $2`
do

        #GET THE NAME OF THE COG CATEGORY FROM COG_functional_categories.txt


        #COUNT THE NUMBER OF OCCURENCES IN COLUMN 7 OF THE E-MAPPER DATA

done
```

<details>
  <summary>Solution</summary>

```
#GO THROUGH EACH COG 1-LETTER CODE AND PRINT OUT NAME AND COUNT FROM EGG NOG OUTPUT FILE
for COG_code in `cut -f 1 $2`
do

        #GET THE NAME OF THE COG CATEGORY FROM COG_functional_categories.txt
        grep ^$COG_code $2;

        #COUNT THE NUMBER OF OCCURENCES IN COLUMN 7 OF THE E-MAPPER DATA
        cut -f 7 $1 | grep $COG_code | wc -l

done
```
</details>


Now that we have our shell script, let's apply it to examine the functional distribution of genes present in our multidrug resistant Klebsiella genome (PCMP_H183) and an environmental Klebsiella genome (ERR025151).

```
#Run shell script on PCMP_H183 and redirect output to file
./COG_code_counter.sh PCMP_H183_eggnog.emapper.annotations COG_functional_categories.txt > PCMP_COG_counts

#Run shell script on ERR025151 and redirect output to file
./COG_code_counter.sh ERR025151_eggnog.emapper.annotations COG_functional_categories.txt > ERR_COG_counts

#To see them side by side, let's use the paste command
paste PCMP_COG_counts ERR_COG_counts
```

Do you see any big differences in how genes are allocated to functional groups between these two genomes?


Perform pan-genome analysis with [Panaroo](https://github.com/gtonkinhill/panaroo)
----------------------------------------


Panaroo is a pan genome pipeline, which takes annotated assemblies in GFF3 format and determines different sets of genes in the dataset i.e Pangenome. The pan-genome is just a fancy term for the full complement of genes in a set of genomes. 

The way Panaroo works is:

1) It gets all the coding sequences from the input GFF files, converts them into protein, and creates pre-clusters of all the genes using CD-HIT.
2) Then, it performs various refinements to reduce annotation errors such as misannotations and finds paralogs.
3) Finally, it will take every isolate and order them by presence/absence of genes.

For our analysis today, we will expand from the two genomes above, to a larger set of 8 *Klebsiella pneumoniae* genomes from human and environmental sources. Six of these genomes are from [this paper](https://www.pnas.org/content/112/27/E3574), and the other two are sequences from our lab. Above we saw that despite having very different lifestyles, the overall attribution of genes to different functional categories didn't look especially different. Using panaroo, we will see how different these genomes are in the genes they actually encode. 

> ***i. Generate pan-genome matrix using Panaroo and GFF files***

Make sure you are in class7 directory

```
cd panaroo

```

Let's look at the panaroo command:

```
less panaroo.sbat
```

We wont run Panaroo but instead use the output files to explore the Pan genome.

Output files:

1. `summary_statistics.txt`: This file is an overview of your pan genome analysis showing the number of core genes(present in all isolates) and accessory genes(genes absent from one or more isolates or unique to a given isolate). 

2. `gene_presence_absence.csv`: This file contain detailed information about each gene including their annotations which can be opened in any spreadsheet software to manually explore the results. It contains plethora of information such as gene name and their functional annotation, whether a gene is present in a genome or not, minimum/maximum/Average sequence length etc.

3. `gene_presence_absence.Rtab`: This file is similar to the gene_presence_absence.csv file, however it just contains a simple tab delimited binary matrix with the presence and absence of each gene in each sample. It can be easily loaded into R using the read.table function for further analysis and plotting. The first row is the header containing the name of each sample, and the first column contains the gene name. A 1 indicates the gene is present in the sample, a 0 indicates it is absent.

4. `core_gene_alignment.aln`: a multi-FASTA alignment of all of the core genes that can be used to generate a phylogenetic tree.

> ***ii. Explore pan-genome matrix gene_presence_absence.csv and gene_presence_absence.Rtab using R***

We're going to use information from `gene_presence_absence.csv` and `gene_presence_absence.Rtab`. Let's take a look at these from the Terminal using `less`:

```
# annotation information is here
less -S gene_presence_absence.csv
# presence/absence information
less -S gene_presence_absence.Rtab
```

> ***iii. Explore pan-genome matrix gene_presence_absence.csv and gene_presence_absence.Rtab using R***

Look at the size of the core and accessory genome for our multidrug resistant and environmental isolates.

```
less summary_statistics.txt
```

Despite having similar distributions of COG categories, we see that the genome content is actually quite different, with only 4400 of each Klebsiella's 5500 genes shared among these 8 strains.


<!---
**Read matrices into R, generate exploratory plots and query pan-genome**

Make a directory onto your local system

```
mkdir ~/Desktop/class6
```

Use scp or cyberduck to get `gene_presence_absence.csv` and `gene_presence_absence.Rtab` onto your laptop desktop folder.

```
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/epid582w23_class_root/epid582w23_class/username/class6/panaroo/gene_presence_absence.csv ~/Desktop/class6
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/epid582w23_class_root/epid582w23_class/username/class6/panaroo/gene_presence_absence.Rtab ~/Desktop/class6
```

> ***i. Prepare and clean data***

- Fire up RStudio and read both files in.

```
# read in annotations (only need 3rd column)
annots = read.csv('~/Desktop/class6/gene_presence_absence.csv')[,3]
# read in presence-absence heatmap
pg_matrix = read.delim('~/Desktop/class6/gene_presence_absence.Rtab', row.names = 1, header=TRUE)
```

- Use head, str, dim, etc. to explore the matrix.

> ***ii. Generate exploratory heatmaps.***

- Make a heatmap for the full matrix

```
# load pheatmap
library(pheatmap)

# make heatmap
pheatmap(pg_matrix,col= c('black', 'red'), show_rownames = F)
```

- Make a heatmap for variable genes (present in at least one, but not all of the genomes)

```
# subset matrix to include only variable genes
pg_matrix_subset = pg_matrix[rowSums(pg_matrix > 0) > 0 & rowSums(pg_matrix > 0) < 4 ,] 
# make heatmap of variable genes
pheatmap(pg_matrix_subset,col= c('black', 'red'), show_rownames = F)
```

> ***iii. Query pan-genome***

-  Which genomes are most closely related based upon shared gene content?

We will use the apply function to determine the number of genes shared by each pair of genomes. 

```
apply(pg_matrix_subset, 2, function(x){
  colSums(pg_matrix_subset == x & pg_matrix_subset == 1)
})
```

- What is the size of the core genome?

Lets first get an overview of how many genes are present in different numbers of genomes (0, 1, 2, 3 or 4) by plotting a histogram. Here, we combine hist with rowSums to accomplish this.

```
hist(rowSums(pg_matrix > 0), col="red")
```

Next, lets figure out how big the core genome is (e.g. how many genes are common to all of our genomes)?

```
sum(rowSums(pg_matrix > 0) == 4)
```

- What is the size of the accessory genome?

Lets use a similar approach to determine the size of the accessory genome (e.g. those genes present in only a subset of our genomes).

```
sum(rowSums(pg_matrix > 0) < 4 & rowSums(pg_matrix > 0) > 0)
```

- What types of genes are unique to a given genome?

So far we have quantified the core and accessory genome, now lets see if we can get an idea of what types of genes are core vs. accessory. Lets start by looking at those genes present in only a single genome. 

```
# genes present only in a single genome
unique_annots = annots[rowSums(pg_matrix > 0) == 1]
unique_annots
```

What do you notice about these genes?

- What is the number of hypothetical genes in core vs. accessory genome?

Looking at unique genes we see that many are annotated as “hypothetical”, indicating that the sequence looks like a gene, but has no detectable homology with a functionally characterized gene. 

Determine the fraction of “hypothetical” genes in unique vs. core. 

```
# genes present in all genomes
core_annots = annots[rowSums(pg_matrix > 0) == 4]

# fraction of unique and core hypothetical genes
# unique hypothetical
sum(grepl("hypothetical" , unique_annots)) / sum(rowSums(pg_matrix > 0) == 1)
# core hypothetical
sum(grepl("hypothetical" , core_annots)) / sum(rowSums(pg_matrix > 0) == 4)
```

Why does this make sense?

-->
