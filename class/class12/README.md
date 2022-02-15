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


Exploring the relationship between community- and healthcare-acquired MRSA using phylogenetic analysis
------------------------------------------------------------------------------------------------------


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
