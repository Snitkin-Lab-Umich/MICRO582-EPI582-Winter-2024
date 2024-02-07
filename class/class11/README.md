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
library(phytools)

#Read in DNA alignment and annotations
mrsa_aln <- read.dna('class12/MRSA_USA300_var_aln.fa',
                     format = "fasta")

annot <- read.table('class12/MRSA_USA300_annot.txt',
                    row.names = 1,
                    header = T)


#Check out the format of our variables
class(mrsa_aln)
str(mrsa_aln)
mrsa_aln

class(annot)
str(annot)


#Build NJ tree
#Create a distance matrix from 
dist_mat <- dist.dna(mrsa_aln)

#Create a neighbor joining tree from distance matrix
nj_tree = nj(dist_mat)

#Midpoint root tree
nj_tree_rooted <- midpoint.root(nj_tree)


#Plot tree with colored tips
#Create vector linking colors to HA/CA
cols = structure(c('blue', 'red'), names = unique(annot$SOURCE))

#Plot tree
#plot(nj_tree_rooted, label.offset = 0.001, cex = 0.5)
plot(nj_tree_rooted, type = "fan", label.offset = 0.001, cex = 0.5)

#Add colors to tips
tiplabels(col = cols[annot[nj_tree_rooted$tip.label,]], pch = 16, frame = "none")

#Add legend
legend('bottomleft', legend = names(cols), 
       col = "black", pt.bg = cols, pch = 21, cex = 1)
```

Based on the tree - do you think there is evidence of an HA-lineage of USA300?

<details>
  <summary>Solution</summary>  
  
  If there were an HA-lineage of USA300 we would expect that all the HA isolates would group together on the tree and share a common ancestor dating back to the
  emergence of this HA-linage. However, the intermixing of CA and HA isolates on the tree, indicates that there is a single lineage of USA300 capable of causing
  infections in both settings. [Based on some work our group has done with a collaborator](https://pubmed.ncbi.nlm.nih.gov/28486667/), we hypothesize that the
  uptick in HA infections is not neccesarily due to increased transmission in healthcare settings, but rather due to an increased prevalence in the community and
  patients transitioning from colonization to infection in the hospital (i.e. asymptomatically colonized on admission, but only show symptoms of infection later
  in their stay).

</details>


Tracking the origin of an blaNDM ST147 Klebsiella pneumoniae outbreak using phylogenetic analysis
-------------------------------------------------------------------------------------------------
For our second phylogenetic exercise we are going to try and understand the origin of an outbreak of NDM-containing /Klebsiella pneumoniae/ in Chicago-area hospitals. Point-prevalence studies for carbapenem-resistant organisms are routinely performed in the Chicago-region due to high prevelence of KPC-containing organisms. However, a few years ago NDM began being observed and then showed a big spike in a handful of skilled-nursing facilities. We undertook a genomic epidemiology investigation to understand the basis for this uptick. Initial sequence analysis revealed that most isolates were from a single sequence type - ST147 of /Klebsiella pneumoniae/. To understand whether this NDM containing organism was imported into the region (once or multiple times) or if perhaps a circulating ST147 strain picked up the NDM gene locally, we performed a phylogenetic analysis including the outbreak isoaltes along with all publically available ST147 genomes. Based on the clustering of our outbreak genomes, as well as the pattern of NDM on the phylgoeny, we hoped to improve our understanding of the outbreak. Below is R code to do the following:

1) Read in a maximum likelihood phylogeny of outbreak and public isolates that we created using reference based variant calling and the too IQTREE
2) Read in meta-data describing the location of origin and whether an isolate harbored NDM
3) Plot the phylogenetic tree
4) Plot next to the tree a heatmap showing location and NDM status of each isolate

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

Based on the tree - what can we infer about the origins of ST147-NDM containing isoaltes in Chicago?

<details>
  <summary>Solution</summary>  
Placing our outbreak genomes in the context of public genomes revealed several interesting things:
  
1) First, the Chicago isolates for a close cluster on the tree, supporting a single introduction followed by regional spread. We followed up on this observation by performing more fine-grained genetic distance and phylogenetic analyses, supporting this hypothesis.
 
2) Second, we observe on the phylogeny that NDM appears to have been acquired multiple times independently in ST147, as evidenced by the discrete clusters observed across the globe.
  
3) Third, we observe that within our outbreak there exists close genetic neighbors of the outbreak strain that do not carry NDM (they actually carry KPC). This supports an NDM-containing plasmid potentially having been acquired by circulating ST147 in the region, and going on to cause a regional outbreak. We performed some detailed analysis of plasmid carraige to confirm this hypothesis.
  
For more details on the analysis check out [our manuscript](https://academic.oup.com/cid/article/73/8/1431/6277037?login=true).
  
</details>
