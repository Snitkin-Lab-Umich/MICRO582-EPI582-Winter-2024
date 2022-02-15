Class 12 – Plotting and interpretted phylogenetic trees
=======================================================

Goals
----
- Learn how to work with DNA sequence data in R
- Learn how to create Neighbor Joining trees in R
- Learn how to visualize phylogenetic trees with associated meta-data in R
- Learn how to interpret phylogenetic trees in the context of an epidemiologic question

R packages for phylogenetic analysis
-------------------------------------
During this session we will exclusively be working with the ape package, which has a multitude of functionality for working with sequence data, trees and making corresponding visualizations. However, there are many other useful packages in R that are worth a look!

A few noteworthy R packages:
1. [ape](https://cran.r-project.org/web/packages/ape/ape.pdf)
2. [phytools](https://cran.r-project.org/web/packages/phytools/index.html) (phylogenetic functions and tree visualization)
3. [ggtree](https://guangchuangyu.github.io/software/ggtree/) (package for advanced tree visualization)
4. [phangorn](https://cran.r-project.org/web/packages/phangorn/index.html) (phylogenetic functions and tree visualization)


Setup
-----
We are going to be working in RStudio again today. Take the following steps to get ready for the lab:

1. Start up your epid582 Rproject and create a new directory in it called class12 to hold data we will be analyzing today. 
2. Go on to Great Lakes and copy over the class 12 files to your working directory
3. Use cyberduck to bring the files down to the class12 directory you created on your own computer


Exploring the relationship between community- and healthcare-acquired MRSA using phylogenetic analysis
------------------------------------------------------------------------------------------------------
For our first phylogenetics exercise, we are going to test the epidemiologic hypothesis that there are different strains of MRSA that are causing hospital-associated (HA) versus community-associated (CA) infections. As a little background, up until the early 2000's methicillin-resistant Staphylococcus aureus (MRSA) infections were almost exclusively restricted to healthcare settings. Then, in the early 2000's a new strain of MRSA burst on the sceen called USA300, which was capable of causing infections in otherwise healthy individuals in the community, outside of healthcare settings. While initially only seeming to occur in the community, over the next several years USA300 was observered to cause infections in hospitals. Some hypothesized that this might be because of an evolutionary event in USA300 that created a new sublineage that was better adapted for spread in hospitals. 

Here, we are going to test this hypothesis by creating a whole-genome phylogeny for a panel of MRSA genomes from a single healthcare center. Half of our genomes are community-associated (i.e. infections on admission to the hospital) and the other half are hosiptal-associated (i.e. infection after 3 days of admission. Below is R code we will go through to do the following:

1) Read in a DNA alignment of MRSA genomes created by a read mapping pipeline
2) Compute a pairwise distance matrix using the dist.dna function
3) Build a neighbor joining tree from the distance matrix using the nj function
4) Visualize our phylogenetic tree, along with CA/HA labels to see if we detect evidence of an HA USA300 strain responsible for healthcare infections

```
#Read in needed R packages
library(ape)

#Read in DNA alignment and annotations
mrsa_aln <- read.dna('MRSA_USA300_var_aln.fa',
                     format = "fasta")

annot <- read.table('MRSA_USA300_annot.txt',
                    row.names = 1,
                    header = T)


#Check out the format of our variables
class(mrsa_aln)
str(mrsa_aln)
mrsa_aln

class(annot)
str(annot)


#Create a distance matrix from 
dist_mat <- dist.dna(mrsa_aln)

#Create a neighbor joining tree from distance matrix
nj_tree = nj(dist_mat)

nj_tree_rooted <- midpoint.root(nj_tree)

#Plot with heatmap
plot(nj_tree_rooted, x.lim = 0.075)
phydataplot(as.matrix(annot),  nj_tree_rooted, style = "m", offset = 0.01)

#Plot with colored tips
plot(nj_tree, label.offset = 0.001)

#plot(nj_tree, type = "fan", label.offset = 0.001)

cols = structure(c('red', 'blue'), names = unique(annot$SOURCE))
tiplabels(col = cols, pch = 16)
```

Based on the tree - do you think there is evidence of an HA-lineage of USA300?

<details>
  <summary>Solution</summary>  
  
```
If there were an HA-lineage of USA300 we would expect that all the HA isolates would group together on the tree and share a common ancestor dating back to the emergence of this HA-linage. However, the intermixing of CA and HA isolates on the tree, indicates that there is a single lineage of USA300 capable of causing infections in both settings. [Based on some work our group has done with a collaborator](https://pubmed.ncbi.nlm.nih.gov/28486667/), we hypothesize that the uptick in HA infections is not neccesarily due to increased transmission in healthcare settings, but rather due to an increased prevalence in the community and patients transitioning from colonization to infection in the hospital (i.e. asymptomatically colonized on admission, but only show symptoms of infection later in their stay).
```

</details>


Tracking the origin of an blaNDM ST147 Klebsiella pneumoniae outbreak using phylogenetic analysis
-------------------------------------------------------------------------------------------------

```
#Read in libraries
library(RColorBrewer)
library(ape)

#Read in tree
tree <- read.tree('class12/st147_outbreak_tree.tree')

#Read in meta-data
meta_data <- read.table('class12/st147_genome_metadata.txt', 
                        sep = "\t",
                        header = T)

#Set color pallette
cols = brewer.pal(8, 'Set1');
f <- function(n) c(cols)

#Plot tree
plot(tree, use.edge.length = T, show.tip.label = F, cex = 0.25,  x.lim = 0.0001)

#Plot heatmap with NDM status and location next to tree
phydataplot(as.matrix(meta_data),  
            tree, 
            style = "m", 
            width = 0.00001, 
            offset = 0.000001, 
            legend = 'side', 
            lwd = 0.1,
            funcol = f)
```
