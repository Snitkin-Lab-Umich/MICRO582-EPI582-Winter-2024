Class 12 – Bacterial genome-wide association studies
====================================================

Goals
----
- Learn how to load in your genotype, phenotype and tree data into R
- Learn how to visualize and interpret your phenotype data overlaid on the tree
- Learn how to run the treeWAS bGWAS tool in R
- Learn how to work with and interpret the results of treeWAS

Background on bGWAS
-------------------
The goal of bacterial genome-wide association studies (bGWAS) is to identify genetic variation associated with some phenotype of interest. In the context of infectious diseases, phenotypes of interest include things like antibiotic resistance, virulence and transmissability. With an understanding of the genetic basis for these phenotypes, we would be able to more effectively survey for high-risk strains, and also devise therapeutics interfering with organisms ability to evade treatment, cause serious infections and spread.

The field of GWAS was pioneered in the context of identifying genetic associations for human disease and phenotypic variation. However, there are significant differences in bacterial and human genome evolution that prevent us from just directly applying tools from human GWAS. Of greatest consequence is the lack of assortment of alleles through sexual reproduction, resulting in strong linkage among genetic variants. The challenge this creates is that it can be difficult to distinguish genetic variation causing phenotypic change, from passenger mutations that arose around the same time and whose linkage is maintained throughout evolution of the descendents.

However, while the nature of bacterial evolution creates challenges for GWAS, it also presents opportunities. Some unique features of bacteria relative to humans are extremely large population sizes and short generation times. Combined with often strong selective pressure for the emergence of traits of interest (e.g. antibiotic resistance, transmissability), it is often observed that traits emerge multiple times in the evolution of a given lineage. This convergence of phenotypic evolution essentially breaks the linkage problem inherent to bacteria, because causal genetic variation is arising independently in different genetic backgrounds. By and large, it is this covergence that is leveraged by bGWAS methods to identify genetic variation that co-occurs with the repeated emergence of phenotypes. 

The treeWAS R package
---------------------
[TreeWAS](https://github.com/caitiecollins/treeWAS) is an R package that is commonly applied for bGWAS analysis. Included in treeWAS are three different association tests that attempt to quantify different ways that a genotype and phenotype can be associated. Today we will focus on the synchonous test, which explicitly evaluates whether the emergence of a genetic variant on a phylogeny co-occurs with the emergence of a phenotype more than would be expected by chance. Put differently,the synchronous tests recreates the evolution of the phenotype and each genetic variant on a phylogeny, and tests whether a variant and phenotype show a strong correlation in when they arise. If a given genetic variant always arises at the same time as a phenotype, that supports a potential causal association.

Setup
-----
We are going to be working in RStudio again today. Take the following steps to get ready for the lab:

1. Start up your epid582 Rproject and create a new directory in it called class13 to hold data we will be analyzing today. 
2. Go on to Great Lakes and copy over the class 13 files to your working directory
3. Use cyberduck to bring the files down to the class13 directory you created on your own computer

Identifying genes associated with clindamycin resistance in USA300 MRSA
-----------------------------------------------------------------------
To test out treeWAS we are going to perform a bGWAS analysis to search among all the genes in the MRSA pan-genome to see if we can identify the gene(s) confering resistance to the antibiotic clindamycin. The input to our bGWAS analysis include:

1) Panaroo matrix - recall that panaroo takes as input a set of genomes and determines the pangenome (i.e. which genes are present/absent in each input genome). 
2) Clindamycine resistance - our phenotype is a binary vector indicating resistance or susceptability to clindamycin
3) Phylogenetic tree - a phylogenetic tree of input MRSA genomes was constructed using IQTREE, a maximum likelihood algorithm

So, when we run treeWAS, we will be evaluating each gene in the input panaroo matrix to assess whether it's gain is associated with the gain of clindamycin resistance more than we would expect by chance.

When running treeWAS several R variables and plots are created. The plots include:
1. A Manhattan plot, which shows the significance of hits (y-axis) across the genome (x-axis). Note that since the Panaroo matrix is in no particular order, the ordering on the x-axis won't have meaning for us, but we can still visually see how many hits there are and their relative strength
2. A histogram of scores for significant hits, along with a line indicating the significance threshold. 

In addition to plots, treeWAS creates rich output variables that are described in detail in the manual. By parsing these output variables you can identify the significantly associated genes, along with their scores and p-values.

Below is the R commands we will employ to run treeWAS and explore the results.

```
### Install and load libraries
install.packages("devtools", dep=TRUE)
library(devtools)

### Install and load libraries
install_github("caitiecollins/treeWAS", build_vignettes = TRUE)
library(treeWAS)


### Read in Pangenome gene presence/absence matrix from Panaroo
geno_df <- read.table(file="class13/geno.tsv", 
                      header = TRUE)

### Read in Phyogenetic tree.
tree <- read.tree(file = "class13/cdc_tree_rooted.tree")


###  Read in binary Phenotype data - whether a MRSA sample 
###  is clindamycin resistant(1) or not(0)
phen_df <- read.table(file="class13/pheno.tsv", 
                      header = TRUE)

#Create phen vector inputted according to treeWAS format
phen <- as.vector(unlist(phen_df$clinda))
names(phen) <- phen_df$samples


###Plot antibiotic resistance on the tree
#Make histogram of resistance and then plot on tree
hist(phen, col = c("red", "blue"))

#Plot tree with antibiotic resistance next to it
plot(tree,
     show.tip.label = F,
     cex = 0.25,
     x.lim = 0.0005)

phydataplot(phen,
            tree,
            style = "m",
            width = 0.00001,
            offset = 0.00001,
            legend = "side",
            lwd = 0)


### Run treeWAS with default parameters - takes 4.5 minutes to finish on local system
out <- treeWAS(snps = t(geno_df), 
               phen = phen, 
               tree = tree, 
               test = 'simultaneous',
               seed = 1)


### Print significant loci associated with the clindamycin resistance
print(out, sort.by.p=FALSE) 


### Plot tree with antibiotic resistance and hits
#Add clindamycin resistance to heatmap dataframe
out_heatmap <- subset(phen_df,select = c('clinda'))

#Add row names
row.names(out_heatmap) <- phen_df$samples 

#Change from binary to Clinda S/R
out_heatmap[out_heatmap == 1] <- 'Clinda R'
out_heatmap[out_heatmap == 0] <- 'Clinda S'

#Add significantly associated genes
sig.hits <- row.names(out$simultaneous[['sig.snps']])
out_heatmap[,sig.hits] <- t(geno_df[sig.hits,row.names(out_heatmap)])

#Plot tree and heatmap!
plot(tree,
     show.tip.label = F,
     cex = 0.25,
     x.lim = 0.0005)

phydataplot(as.matrix(out_heatmap),
            tree,
            style = "m",
            width = 0.000025,
            offset = 0.00001,
            legend = "side",
            lwd = 0)
```


Ultimtely, our association picked up three genes, of which two are the most strongly associated. These two genes are [ermC](https://www.uniprot.org/uniprot/P13978), which is an enzyme that modifies the 23S rRNA that is the target of clindamycin, and has shown to confer [resistance in Staph aureus](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4411429/). The second gene is carried on the small plasmid that harbors the ermC gene, which explains it's strong association.
