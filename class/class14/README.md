Class 14 - Whole genome alignment and Recombination Detection
============================================================

Goal
----

- In this section, we will employ Parsnp to align three hospital outbreak Acinetobacter genomes.
- Create a multiple sequence alignment and visualize them in Gingr. 
- Perform recombination filtering with Gubbins and see how recombination can distort phylogenetic signal.

![phylo](phylo.png)

The backstory on these acinetobacter genomes is that Abau_A, Abau_B and Abau_C are representatives of three clones (as defined by pulsed-field gel electrophoresis - a low-resolution typing method) that were circulating in our hospital. 

One of the goals of our published study was to understand the relationship among these clones to discern whether: 

1) the three clones represent three independent introductions into the hospital or 

2) the three clones originated from a single introduction into the hospital, with subsequent genomic rearrangement leading to the appearance of unique clones. 

The types of phylogenetic analyses you will be performing here are the same types that we used to decipher this mystery.

The other two genomes you will be using are **ACICU and AB0057.** 

**ACICU** is an isolate from a hospital in France, and its close relationship to our isolates makes it a good reference for comparison. 

**AB0057** is a more distantly related isolate that we will utilize as an out-group in our phylogenetic analysis. The utility of an out-group is to help us root our phylogenetic tree, and gain a more nuanced understanding of the relationship among strains.

Execute the following command to copy files for todays class:

```
wd

cp -r /scratch/epid582w22_class_root/epid582w22_class/shared_data/data/class13 ./

```


Perform Whole genome alignment with [Parsnp](https://harvest.readthedocs.io/en/latest/content/parsnp.html)
----------------------------------------------------------------------------------------------------------

An alternative approach for identification of variants among genomes is to perform whole genome alignments of assemblies. If the original short read data is unavailable, this might be the only approach available to you. Typically, these programs don’t scale well to large numbers of genomes (e.g. > 100), but they are worth being familiar with. We will use the tool mauve for constructing whole genome alignments of our five A. baumannii genomes.

> ***i. Perform genome alignment with Parsnp***

Create a conda environment class13 that will install Parsnp/Harvesttools/Gubbins for you. 

```
conda create -n class13 -c bioconda parsnp harvesttools gubbins

conda activate class13

# invoke parsnp and harvesttool's help menu to check if it was installed properly
parsnp -h

harvesttools -h

run_gubbins.py -h

cd class13
```

Now we will ask parsnp to align all the genomes in Abau_genomes directory and also ask parsnp to use Reference_genome/ACICU_genome.fasta as a reference genome.

```
# Runtime 0m42.461s
parsnp -c -d Abau_genomes/ -r Abau_genomes/Reference_genome/ACICU_genome.fasta -o parsnp_results -p 2 -v

```

Parsnp will generate various output files in parsnp_results folder:

- Newick formatted core genome SNP tree: parsnp_results/parsnp.tree
- Gingr formatted binary archive: parsnp_results/parsnp.ggr
- XMFA formatted multiple alignment: parsnp_results/parsnp.xmfa

> ***ii. Convert ginger formatted binary file to fasta format to use for Gubbins***

We will use harvesttools to convert parsnp.ggr to a multi-fasta alignment output (concatenated LCBs) file - parsnpLCB.aln that we will use later as input for Gubbins.

```
cd parsnp_results

harvesttools -i parsnp.ggr -M parsnpLCB.aln
```

> ***iii. View multiple genome alignment in gingr***

Gingr is a visualization tool that accompanies parsnp. So, let's download the files we created using parsnp and load them into gingr.

Go to your local system terminal or open a new terminal tab and run these commands to create a new directory and copy files from great lakes to your local Desktop using Cyberduck.

```
mkdir ~/Desktop/Abau_parsnp

cd ~/Desktop/Abau_parsnp

```

Drag and drop parsnp.ggr, parsnpLCB.aln, parsnp.tree to this folder.


Now, fire up gingr and use File->open tab to read in parsnp.ggr

Notice the structure of the tree (i.e. which genomes are closely related to one another) and whether variants are or aren't evenly spaced across the genome.


Phylogenetic analysis in [APE](http://ape-package.ird.fr/)
----------------------------------------------------------

We did initial visualization of parsnp output files in gingr, now we are going to do further exploration of DNA alignments in R. 

First we will determine how many variants separate each genome, second we will visualize the phylogenetic tree produced by parsnp and finally we will look for evidence of recombination among our genomes.

> ***i. Read alignment into R***

Fire up RStudio, set your working directory to ~/Desktop/Abau_parsnp/ or wherever you have downloaded your parsnp files and install/load ape

Use the read.dna function in ape to read in you multiple alignments. 
Print out the variable to get a summary.

```
#SET YOUR DIRECTORY
setwd("~/Desktop/Abau_parsnp/")

library(ape)

#READ IN THE MULTIPLE GENOME ALIGNMENT AND CHANGE THE NAMES TO REMOVE FILE EXTENSIONS
abau_msa = read.dna('parsnpLCB.aln', format = "fasta") 
row.names(abau_msa) = gsub(".fa|.fasta", "", row.names(abau_msa))
```

> ***ii. Get variable positions***

The DNA object created by read.dna can also be addressed as a matrix, where the columns are positions in the alignment and rows are your sequences. We will next treat our alignment as a matrix, and use apply and colSums to get positions in the alignment that vary among our sequences. Examine these commands in detail to understand how they are working together to give you a logical vector indicating which positions vary in your alignment.

```

abau_msa_bin = apply(abau_msa, 2, FUN = function(x){x == x[1]}) 

abau_var_pos = colSums(abau_msa_bin) < 5
```

> ***iii. Get non-gap positions***

For our phylogenetic analysis we want to focus on the core genome, so we will next identify positions in the alignment where all our genomes have sequence.

```
non_gap_pos = colSums(as.character(abau_msa) == '-') == 0
```

> ***iv. Count number of variants between sequences***

Now that we know which positions in the alignment are core and variable, we can extract these positions and count how many variants there are among our genomes. To count pairwise variants we will use the dist.dna function in ape. The model parameter indicates that we want to compare sequences by counting differences. Print out the resulting matrix to see how different our genomes are.

```

abau_msa_var = abau_msa[,abau_var_pos & non_gap_pos ]
var_count_matrix = dist.dna(abau_msa_var, model = "N")

```

Examining the pairwise distances among our isolates, we see that our genomes have thousands of variants between them. Based on the estimated evolutionary rate of Acinetobacter, we would expect this amount of variation to take hundreds of years to accumulate via mutation and vertical inherentence. Thus, based on this observation it seems unlikely that these are related by recent transmission. However, before we reach our final conclusion we need to assess whether all of this variation was due to mutation and vertical inheretance, or if some of the variation is due to horizontal transfer via recombination.

> ***vi. View phylogenetic tree***

First, let's read in the tree produced by parsnp and plot it using ape.

```
parsnp_tree = read.tree('parsnp.tree')
plot(parsnp_tree)
```

Next, let's root our tree by the outgroup so that the structure is correct.

```
parsnp_tree$tip.label<-gsub("\'","",parsnp_tree$tip.label)
parsnp_tree_rooted = root(parsnp_tree, "Abau_AB0057_genome.fa")
plot(parsnp_tree_rooted)
```

Now that the tree is rooted, let's drop the outgroup so we can more clearly see the tree structure for our isolates of interest.

```
parsnp_tree_rooted_drop = drop.tip(parsnp_tree_rooted, c('Abau_AB0057_genome.fa', 'ACICU_genome.fasta.ref'))
plot(parsnp_tree_rooted_drop)
```

Notice that with this tree, genomes A and B are more closely related, suggesting that they share a more recent common ancestor than C. In the realm of genomic epidemiology we would infer that A and B are more closely related in a putative transmission chain. Let's see if the tree structure, and therefore our understanding of the outbreak is influenced by filtering out recombinant regions.


Perform SNP density analysis to discern evidence of recombination
-----------------------------------------------------------------

An often-overlooked aspect of a proper phylogenetic analysis is to exclude recombinant sequences. Homologous recombination in bacterial genomes is a mode of horizontal transfer, wherein genomic DNA is taken up and swapped in for a homologous sequence. The reason it is critical to account for these recombinant regions is that these horizontally acquired sequences do not represent the phylogenetic history of the strain of interest, but rather in contains information regarding the strain in which the sequence was acquired from. One simple approach for detecting the presence of recombination is to look at the density of variants across a genome. The existence of unusually high or low densities of variants is suggestive that these regions of aberrant density were horizontally acquired. Here we will look at our closely related A. baumannii genomes to see if there is evidence of aberrant variant densities.

> ***i. Subset sequences to exclude the out-group***

For this analysis we want to exclude the out-group, because we are interested in determining whether recombination would hamper our ability to reconstruct the phylogenetic relationship among our closely related set of genomes.  

- Note that the names of the sequences might be different for you, so check that if the command doesn’t work. Try printing out 
abau_msa_no_outgroup to check that it worked.

```
abau_msa_no_outgroup = abau_msa[c('ACICU_genome.ref','AbauA_genome','AbauB_genome','AbauC_genome'),]

```

> ***ii. Get variable positions***

Next, we will get the variable positions, as before

```

abau_msa_no_outgroup_bin = apply(abau_msa_no_outgroup, 2, FUN = function(x){x == x[1]}) 

abau_no_outgroup_var_pos = colSums(abau_msa_no_outgroup_bin) < 4

```

> ***iii. Get non-gap positions***

Next, we will get the core positions, as before

```

abau_no_outgroup_non_gap_pos = colSums(as.character(abau_msa_no_outgroup) == '-') == 0

```

> ***iv. Create overall histogram of SNP density***

Finally, create a histogram of SNP density across the genome. 

```
hist(which(abau_no_outgroup_var_pos & abau_no_outgroup_non_gap_pos), 10000)
```

Does the density look even, or do you think there might be just a touch of recombination?

Perform recombination filtering with [Gubbins](https://www.google.com/search?q=gubbins+sanger&ie=utf-8&oe=utf-8)
----------------------------------------------

Now that we know there is recombination, we know that we need to filter out the recombinant regions to discern the true phylogenetic relationship among our strains. In fact, this is such an extreme case (~99% of variants of recombinant), that we could be totally misled without filtering recombinant regions. To accomplish this we will use the tool gubbins, which essentially relies on elevated regions of variant density to perform recombination filtering.

> ***i. Run gubbins on your fasta alignment***

Go back on great lakes and activate class13 environment

```
conda activate class13
```

Run gubbins on your fasta formatted alignment

```
wd

cd class13

cd parsnp_results

#Runtime 120.24 s
run_gubbins.py --filter_percentage 50 --outgroup Abau_AB0057_genome.fa parsnpLCB.aln

```

> ***ii. View recombination regions detected by gubbins in Phandango***

Phandango is a web based tool that is useful for visualizing output from many common microbial genomic analysis programs. Here we will use it to visualize the recombination regions detected by gubbins. 

First let's download a summary of recombinant regions in gff format onto your local system. Use cyberduck to drag and drop these files to ~/Desktop/Abau_parsnp - parsnpLCB.recombination_predictions.gff, parsnpLCB.node_labelled.final_tree.tre 


Next, go the the phandango website (https://jameshadfield.github.io/phandango/#/), and just drag the gff file - parsnpLCB.recombination_predictions.gff and parsnpLCB.node_labelled.final_tree.tre into your web browser. 

Does gubbins seem to have identified recombinant regions where we saw elevated variant density? In addition, which genomes seem to share the most recombinant regions?


Finally, lets look at the recombination-filtered tree to see if this alters our conclusions. 

To view the tree we will use the ape package in R:

```

# In RStudio

# Load ape library
library(ape)

# Path to tree file
tree_file <- '~/Desktop/Abau_parsnp/parsnpLCB.node_labelled.final_tree.tre'

# Read in tree
gubbins_tree <- read.tree(tree_file)

# Drop the outgroup for visualization purposes
gubbins_tree_noOG = drop.tip(gubbins_tree, c('Abau_AB0057_genome.fa'))

plot(gubbins_tree_noOG)

```

How does the structure look different than the unfiltered tree?

- Note that turning back to the backstory of these isolates, Abau_B and Abau_C were both isolated first from the same patient. So this analysis supports that patient having imported both strains, which likely diverged at a prior hospital at which they resided.


