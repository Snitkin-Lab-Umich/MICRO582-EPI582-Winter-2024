Class 13 – Recombination detection
==================================

Goal
----

On day 1, we ran through a pipeline to map reads against a reference genome and call variants, but didn’t do much with the variants we identified. Among the most common analyses to perform on a set of variants is to construct phylogenetic trees. 
- In this section, we will employ Parsnp to align genomes and create a multiple sequence alignment and visualize how they align in Gingr. 
- Explore different tools for generating and visualizing phylogenetic trees.
- Perform recombination filtering with Gubbins and see how recombination can distort phylogenetic signal.

![phylo](phylo.png)

For the first several exercises, we will use the A. baumannii genomes that we worked with yesterday afternoon. 
The backstory on these genomes is that Abau_A, Abau_B and Abau_C are representatives of three clones (as defined by pulsed-field gel electrophoresis - a low-resolution typing method) that were circulating in our hospital. 

One of the goals of our published study was to understand the relationship among these clones to discern whether: 

1) the three clones represent three independent introductions into the hospital or 

2) the three clones originated from a single introduction into the hospital, with subsequent genomic rearrangement leading to the appearance of unique clones. 

The types of phylogenetic analyses you will be performing here are the same types that we used to decipher this mystery.
The other two genomes you will be using are ACICU and AB0057. ACICU is an isolate from a hospital in France, and its close relationship to our isolates makes it a good reference for comparison. AB0057 is a more distantly related isolate that we will utilize as an out-group in our phylogenetic analysis. The utility of an out-group is to help us root our phylogenetic tree, and gain a more nuanced understanding of the relationship among strains.

Execute the following command to copy files for this afternoon’s exercises to your scratch directory:

```
wd

#or

cd /scratch/micro612w21_class_root/micro612w21_class/username

cp -r /scratch/micro612w21_class_root/micro612w21_class/shared/data/day3am ./

```

<!---commenting out Mauve and switching it to Parsnp 2021-04-15
Perform whole genome alignment with [Mauve](http://darlinglab.org/mauve/mauve.html) and convert alignment to other useful formats
-------------------------------------------
[[back to top]](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3aming/README.md)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

An alternative approach for identification of variants among genomes is to perform whole genome alignments of assemblies. If the original short read data is unavailable, this might be the only approach available to you. Typically, these programs don’t scale well to large numbers of genomes (e.g. > 100), but they are worth being familiar with. We will use the tool mauve for constructing whole genome alignments of our five A. baumannii genomes.

> ***i. Perform mauve alignment and transfer xmfa back to great lakes***

Use cyberduck/scp to get genomes folder Abau_genomes onto your laptop

```
Run these commands on your local system/terminal:

cd ~/Desktop (or wherever your desktop is) 

mkdir Abau_mauve

cd Abau_mauve 

- Now copy Abau_genomes folder residing in your day3am folder using scp or cyberduck:


scp -r username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/Abau_genomes ./


```

Run mauve to create multiple alignment

```

i. Open mauve 
ii. File -> align with progressiveMauve 
iii. Click on “Add Sequnce” and add each of the 5 genomes you just downloaded(under Abau_genomes folder)
iv. Name the output file “mauve_ECII_outgroup” and make sure it is in the directory you created for this exercise i.e Abau_mauve
v. Click Align! 
vi. Wait for Mauve to finish and explore the graphical interface

```

Use cyberduck or scp to transfer your alignment back to great lakes for some processing

```


scp ~/Desktop/Abau_mauve/mauve_ECII_outgroup username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am 

```


```
# Change directory to Abau_genomes under day3am that contains input fasta files for alignment
cd day3am/Abau_genomes/

# Add command line mauve to your PATH variable
export PATH=/scratch/micro612w21_class_root/micro612w21_class/shared/bin/mauve/linux-x64/:$PATH

progressiveMauve --output=mauve_ECII_outgroup ACICU_genome.fasta AbauA_genome.fasta AbauB_genome.fasta AbauC_genome.fasta Abau_AB0057_genome.fa
```

> ***ii. Convert alignment to fasta format***

Mauve produces alignments in .xmfa format (use less to see what this looks like), which is not compatible with other programs we want to use. We will use a custom script convert_msa_format.pl to change the alignment format to fasta format


```
# Now run these command in day3am folder on great lakes:

module load Bioinformatics

module load bioperl/1.7.2

conda activate micro612

perl ../convert_msa_format.pl -i mauve_ECII_outgroup -o mauve_ECII_outgroup.fasta -f fasta -c

sed -i 's/.fa.*//g' mauve_ECII_outgroup.fasta 

```
commenting out Mauve and switching it to Parsnp 2021-04-15
-->




Perform Whole genome alignment with [Parsnp](https://harvest.readthedocs.io/en/latest/content/parsnp.html) and convert alignment to other useful formats
-------------------------------------------------------------------------------------------------------------------------------------------------------

An alternative approach for identification of variants among genomes is to perform whole genome alignments of assemblies. If the original short read data is unavailable, this might be the only approach available to you. Typically, these programs don’t scale well to large numbers of genomes (e.g. > 100), but they are worth being familiar with. We will use the tool mauve for constructing whole genome alignments of our five A. baumannii genomes.

> ***i. Perform genome alignment with Parsnp***

Create a conda environment day3am that will install Parsnp/Harvesttools for you. Run these commands to generate a new conda environment.

```
# Deactivate conda environment if you have loaded it.
conda deactivate

conda env create -f /scratch/micro612w21_class_root/micro612w21_class/shared/data/day3am/day3am.yml -n day3am

conda activate day3am

# invoke parsnp and harvesttool's help menu to check if it was installed properly
parsnp -h

harvesttools -h

cd day3am
```

Now we will ask parsnp to align all the genomes in Abau_genomes directory and also ask parsnp to use Reference_genome/ACICU_genome.fasta as a reference genome.

```

parsnp -c -d Abau_genomes/ -r Abau_genomes/Reference_genome/ACICU_genome.fasta -o parsnp_results -p 2 -v

```

Parsnp will generate various output files in parsnp_results folder:

- Newick formatted core genome SNP tree: parsnp_results/parsnp.tree
- SNPs used to infer phylogeny: parsnp_results/parsnp.vcf
- Gingr formatted binary archive: parsnp_results/parsnp.ggr
- XMFA formatted multiple alignment: parsnp_results/parsnp.xmfa

> ***ii. Convert ginger formatted binary file to fasta format***

We will use harvesttools to convert parsnp.ggr to a multi-fasta alignment output (concatenated LCBs) file - parsnpLCB.aln

```
cd parsnp_results

harvesttools -i parsnp.ggr -M parsnpLCB.aln
```

> ***iii. View multiple genome alignment in gingr***

Gingr is a visualization tool that accompanies parsnp. So, let's download the files we created using parsnp and load them into gingr.

Go to your local system terminal or open a new terminal tab and run these commands to create a new directory and copy files from great lakes to your local Desktop.

```
mkdir ~/Desktop/Abau_parsnp

cd ~/Desktop/Abau_parsnp

# Make sure to replace username with your uniq name
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/parsnp_results/parsnpLCB.aln ./
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/parsnp_results/parsnp.tree ./
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/Abau_genomes/Reference_genome/ACICU.gb ./

```

Now, fire up gingr and use File->open tab to read in these files one by one - .aln, .tree and .gb files. 

Notice the structure of the tree (i.e. which genomes are closely related to one another) and whether variants are or aren't evenly spaced across the genome.


Perform some DNA sequence comparisons and phylogenetic analysis in [APE](http://ape-package.ird.fr/), an R package
------------------------------------------------------------------------

We've done some initial visualization of the parsnp output files in gingr, now we are going to do some further exploration in R. 

First we will determine how many variants separate each genome, second we will visualize the phylogenetic tree produced by parsnp and finally we will look for evidence of recombination among our genomes.

> ***i. Read alignment into R***

Fire up RStudio, set your working directory to ~/Desktop/Abau_parsnp/ or wherever you have downloaded your parsnp files and install/load ape

Use the read.dna function in ape to read in you multiple alignments. 
Print out the variable to get a summary.

```
#SET YOUR DIRECTORY
setwd("~/Desktop/Abau_parsnp/")

#INSTALL THE ape PACKAGE FOR PHYLOGENETIC ANALYSIS
install.packages("ape")
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

<!---commenting out building NJ tree 2021-04-15
Now we are ready to construct our first phylogenetic tree! 

We are going to use the Neighbor Joining algorithm, which takes a matrix of pairwise distances among the input sequences and produces the tree with the minimal total distance. In essence, you can think of this as a distance-based maximum parsimony algorithm, with the advantage being that it runs way faster than if you were to apply a standard maximum parsimony phylogenetic reconstruction.

As a first step we are going to build a more accurate distance matrix, where instead of counting variants, we will measure nucleotide distance using the Jukes-Cantor model of sequence evolution. This is the simplest model of sequence evolution, with a single mutation rate assumed for all types of nucleotide changes.

```
dna_dist_JC = dist.dna(abau_msa, model = "JC")
```

Next, we will use the ape function nj to build our tree from the distance matrix

```
abau_nj_tree = nj(dna_dist_JC)
```

Finally, plot your tree to see how the genomes group.

```
plot(abau_nj_tree)
```
commenting out building NJ tree 2021-04-15
-->

First, let's read in the tree produced by parsnp and plot it using ape.

```
parsnp_tree = read.tree('parsnp.tree')
plot(parsnp_tree)
```

Next, let's root our tree by the outgroup so that the structure is correct.

```
parsnp_tree_rooted = root(parsnp_tree, 'Abau_AB0057_genome.fa')
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

Go back on great lakes and load modules required by gubbins

<!---
Older version:
module load python/2.7.3 biopython dendropy reportlab fasttree RAxML fastml/gub gubbins
-->

```
# Deactivate day3am conda environment
conda deactivate

module load Bioinformatics

module load gubbins/2.3.1

```

Run gubbins on your fasta formatted alignment

```
d3m

cd parsnp_results

run_gubbins -v -f 50 -o Abau_AB0057_genome.fa parsnpLCB.aln

```

> ***ii. View recombination regions detected by gubbins in Phandango***

Phandango is a web based tool that is useful for visualizing output from many common microbial genomic analysis programs. Here we will use it to visualize the recombination regions detected by gubbins. 

First let's download a summary of recombinant regions in gff format onto your local system. Type these commands on your local terminal.

```

cd ~/Desktop/Abau_parsnp

scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/parsnp_results/parsnpLCB.recombination_predictions.gff  ./

scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/parsnp_results/parsnpLCB.node_labelled.final_tree.tre  ./

```

Next, go the the phandango website (https://jameshadfield.github.io/phandango/#/), and just drag the gff file - parsnpLCB.recombination_predictions.gff and your parsnp tree - parsnp.tree into your web browser. 

Does gubbins seem to have identified recombinant regions where we saw elevated variant density? In addition, which genomes seem to share the most recombinant regions?


<!---
Commenting out gubbins drawer 2021-04-15

> ***ii. Create gubbins output figure***

Gubbins produces a series of output files, some of which can be run through another program to produce a visual display of filtered recombinant regions. Run the gubbins_drawer script to create a pdf visualization of recombinant regions. 

The inputs are: 

1) the recombination filtered tree created by gubbins (mauve_ECII_outgroup.final_tree.tre),

2) the pdf file to create (mauve_ECII_outgroup.recombination.pdf) and 

3) a .embl representation of recombinant regions (mauve_ECII_outgroup.recombination_predictions.embl).

```

gubbins_drawer mauve_ECII_outgroup.final_tree.tre mauve_ECII_outgroup.recombination_predictions.embl -o mauve_ECII_outgroup.recombination.pdf

```
> ***iii. Download and view gubbins figure and filtered tree in R***

Use cyberduck or scp to get gubbins output files into Abau_mauve on your local system

```

cd ~/Desktop/Abau_mauve

scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/Abau_genomes/mauve_ECII_outgroup.recombination.pdf  ./
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/Abau_genomes/mauve_ECII_outgroup.final_tree.tre  ./

```

Open up the pdf and observe the recombinant regions filtered out by gubbins. Does it roughly match your expectations based upon your SNP density plots?

commenting out building NJ tree 2021-04-15
-->


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

<!--Create annotated publication quality trees with [iTOL](http://itol.embl.de/)-->

Overlay metadata on your tree using R 
------------------------------------------------------

For the final exercise we will use a different dataset, composed of USA300 methicillin-resistant Staphylococcus aureus genomes. USA300 is a strain of growing concern, as it has been observed to cause infections in both hospitals and in otherwise healthy individuals in the community. An open question is whether there are sub-clades of USA300 in the hospital and the community, or if they are all the same. Here you will create an annotated phylogenetic tree of strains from the community and the hospital, to discern if these form distinct clusters.

> ***i. Download MRSA genome alignment from great lakes***

Use cyberduck or scp to get genomes onto your local system.

```

cd ~/Desktop (or wherever your desktop is) 
mkdir MRSA_genomes 
cd MRSA_genomes

scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/2016-03-09_KP_BSI_USA300.fa  ./
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day3am/HA_vs_CA  ./


```

> ***ii. Look at SNP density for MRSA alignment in R***

Before we embark on our phylogenetic analysis, lets look at the SNP density to verify that there is no recombination. 

Fire up R studio and run these commands.

```
library(ape)
mrsa_msa = read.dna('2016-03-09_KP_BSI_USA300.fa', format = 'fasta') 
mrsa_msa_bin = apply(mrsa_msa, 2, FUN = function(x){x == x[1]}) 
mrsa_var_pos = colSums(mrsa_msa_bin) < nrow(mrsa_msa_bin) 
hist(which(mrsa_var_pos), 10000)

```

Does it look like there is evidence of recombination?

<!-- > ***iii. Create fasta alignment with only variable positions*** -->

<!--Next, lets create a new fasta alignment file containing only the variant positions, as this will be easier to deal with in Seaview-->

<!--```-->

<!--write.dna(mrsa_msa[, mrsa_var_pos], file = '2016-3-9_KP_BSI_USA300_var_pos.fa', format = 'fasta')-->

<!--```-->

> ***iii. Create neighbor joining tree in R ***

We will be using R to construct a neighbor-joining tree. Neighbor joining trees fall under the category of "distance-based" tree building. There are more sophisticated algorithms for tree building, including maximum likelihood and bayesian methods. 

There are a number of bioinformatics tools to create trees. Note that in your research it is not a good idea to use these phylogenetic tools completely blind and I strongly encourage embarking on deeper learning yourself, or consulting with an expert before doing an analysis for a publication. 

One resource here at University of Michigan is the Phylogenetic Methods course, EEB 491. 

First, we will need to install and load in some packages to work with tree objects in R. 
```

install.packages('phangorn')
install.packages('phytools')

library(ape) #installed previously, load in if not already in your environment 
library(phangorn)
library(phytools)
```

Next, we will create a pairwise SNV distance matrix of the MRSA samples to create a neighbor joining tree. 

```
mrsa_msa_var = mrsa_msa[, mrsa_var_pos]
dna_dist = dist.dna(mrsa_msa_var, model = 'N', as.matrix = TRUE)
```

Finally, we can use the distance matrix to construct a neighbor joining tree using the function nj()
```
NJ_tree = nj(dna_dist) 
```

We can look at our tree using plot()

```
plot(NJ_tree)
```

We can explore other representations of our tree in R using the flag "type" in the plot function 
```
plot(NJ_tree, type = 'fan')
plot(NJ_tree, type = 'cladogram')
plot(NJ_tree, type = 'phylogram') #default

```




> ***iv. Add annotations to tree in R ***

Now, we will overlay our data on whether an isolate was from a community or hospital infection onto the tree. 

Here is a great example of some of the different ways to annotate your trees in R: https://rdrr.io/cran/ape/man/nodelabels.html

First, we will read in our metadata. 

```
metadata = read.table('HA_vs_CA', header = TRUE, stringsAsFactors = FALSE)

```

Next, let's clean up the tree tip label names to match the metadata IDs, and then drop the tree tips we don't have metadata for. 
```
NJ_tree$tip.label = gsub('_R.*','',NJ_tree$tip.label)
NJ_tree = drop.tip(NJ_tree, setdiff(NJ_tree$tip.label, metadata$ID))

```

Next, we will create our isolate legend and assign colors to the legend. It's important that the labels are in the same order as the tree tips. So, we will use an sapply statement (which is like a for loop) to iterate through the tree tip labels, and figure out the label for each tree tip id. 

```
isolate_legend = sapply(NJ_tree$tip.label, function(id){
  metadata$SOURCE[metadata$ID == id]
})
isolate_colors = structure(c('blue', 'red'), names = sort(unique(isolate_legend)))

```

Finally, we can use the function tiplabels() to add annotations to our tree and the function legend() to put a legend on our tree.  

```
plot(NJ_tree, type = 'fan', no.margin = TRUE, 
     cex = 0.5, label.offset = 3, align.tip.label = FALSE)
tiplabels(pie = to.matrix(isolate_legend,names(isolate_colors)), 
          piecol = isolate_colors, cex = 0.3, adj = 1.4)
legend('bottomleft', legend = names(isolate_colors), 
       col = "black", pt.bg = isolate_colors, pch = 21, cex = 1)

```




<!--One of the most powerful features of iTOL is its ability to overlay diverse types of descriptive meta-data on your tree (http://itol.embl.de/help.cgi#datasets). Here, we will overlay our data on whether an isolate was from a community or hospital infection. To do this simply drag-and-drop the annotation file (2016-3-9_KP_BSI_USA300_iTOL_HA_vs_CA.txt) on your tree and voila! -->

- Do community and hospital isolates cluster together, or are they inter-mixed?


Note, there are also web tools out there to overlay metadata onto your tree and to manipulate the tree in other ways. One such tool is [iTOL](https://itol.embl.de/). 
