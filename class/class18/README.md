Class 18 – Regional outbreak analysis
====================================================

Goals
----
- Learn how to generate descriptive figures to characterize a regional outbreak across multiple healthcare facilities
- Learn how to interpret descriptive figures to understand where transmission is occuring
- Get exposed to generating figures using ggplot and ggtree

Setup
-----
We are going to be working in RStudio again today. Take the following steps to get ready for the lab:

1. Start up your epid582 Rproject and create a new directory in it called class18 to hold data we will be analyzing today. 
2. Go on to Great Lakes and copy over the class 18 files to your working directory
3. Use cyberduck to bring the files down to the class18 directory you created on your own computer

Package installation and library loading
----------------------------------------
For performing descriptive analyses of a regional outbreak we will use several new R packages. The core package we will use for processing genomic data is called regentrans, and was developed primarily by two trainees in my lab (Zena Lapp and Sophie Hoffman). Regentrans uses a DNA alignment, phylogenetic tree and different types of meta-data to create R variables that can easily feed into R plotting packages (e.g. ggplot, ggtree) to create useful figures for describing regional outbreaks.

A few other notes on packages that we wil be using:
1. In prior sessions we have made plots using base R. Today we will be using ggplot, which is extremely powerful for creating informative figures for observational data. I will not be discussing ggplot in detail, but there are lots of great online resources to learn more!
2. In prior sessions we used ape to plot our trees with meta-data. Today we will ue ggtree, which in my opinion allows for creation of much more aestetically pleasing figures, with options for overlaying different types of meta-data. Again, we will not discuss in detail, but there is great [documentation](https://yulab-smu.top/treedata-book/)

```
#Setup regentrans
devtools::install_github('Snitkin-Lab-Umich/regentrans')

library(regentrans)

#Load other required libraries (install if needed)
library(ape)
library(ggplot2)
library(tibble)
library(tidyverse)
library(ggtree)
```

Background on regional outbreak
-------------------------------



```
#Read in meta-data
st147_meta_data <- read.table('class18/study_isolate_metadata_all.csv',
                              sep = ",",
                              header = T)

#Plot isolate count over time
ggplot(data=st147_meta_data, aes(x=f_id, fill = bla), ylim = 20) +
  geom_bar(stat="count") + 
  facet_grid(factor(st147_meta_data$survey)) +
  theme(axis.text.x = element_text(angle = 90))
```


Calculate pairwise genetic distances and look for evidence of transmission within facilities
--------------------------------------------------------------------------------------------

```
##Make plots of pairwise distances within and between facilities
#Read in alignment
st147_aln <- read.dna('class18/st147_variant_aln.fasta',
                      format = "fasta")

#Get distance matrix
dists <- ape::dist.dna(x = st147_aln, # DNAbin object as read in above
                       as.matrix = TRUE, # return as matrix
                       model = "N", # count pairwise distances
                       pairwise.deletion = TRUE # delete sites with missing data in a pairwise way
)

#Get pairwise distance object from regentrans
st147_meta_data_subset <- subset(st147_meta_data, gID %in% row.names(dists)) #subset to match distance matrix

facils <- structure(st147_meta_data_subset$f_id, names = st147_meta_data_subset$gID) #get isolates named by facility
pts <- structure(st147_meta_data_subset$pt_id, names = st147_meta_data_subset$gID) #get isolates named by patient

pair_types <- get_pair_types(dists = dists, 
                             locs = facils, 
                             pt = pts)
     
#Plot pairwise distances for intra- and inter-facility pairs
ggplot(data = pair_types, aes(x = pairwise_dist, fill = pair_type)) + 
  geom_histogram(position = 'identity', alpha = 0.4, bins = 50) +
  labs(x = "Pairwise SNV distance", y = "Count", fill = "") 
                             
  
#Zoom in on small values to see enrichment for intra-facility at small distances
ggplot(data = pair_types, aes(x = pairwise_dist, fill = pair_type)) + 
  geom_histogram(position = 'identity', alpha = 0.4, bins = 50) +
  labs(x = "Pairwise SNV distance", y = "Count", fill = "") +
  xlim(-1,51)
  
#Plot intra versus inter facility dists over time
surveys <- structure(st147_meta_data_subset$survey, names = st147_meta_data_subset$gID) #get surveys named by isolates

pair_types %>% 
  mutate(surv1 = surveys[isolate1], surv2 = surveys[isolate2]) %>% 
  filter(surv1 == surv2) %>% 
  ggplot(aes(x = pairwise_dist, fill = pair_type)) + 
  geom_histogram(position = 'identity', alpha = 0.4, bins = 50) +
  labs(x = "Pairwise SNV distance", y = "Count", fill = "")+
  facet_grid(~surv1) 
```


```
##Look for evidence of SNV threshold enriched in intra-facility transmission
#Use regentrans to get fraction of intra- versus inter- facility pairs at different
#SNV bins
frac_intra <- get_frac_intra(pair_types = pair_types)

#Plot intra-facility fraction versus SNV distance
ggplot(data =frac_intra, aes(x = pairwise_dist, y = frac_intra)) + 
  geom_bar(stat = "identity", alpha = 0.5) + 
  scale_fill_grey() + 
  labs(x = "Pairwise SNV distance", y = "Fraction of intra-facility pairs") + 
  ylim(0, 1) + xlim(-1,51) 
```

Examine whole-genome phylogeny and extract putative transmission clusters
-------------------------------------------------------------------------

```
##Plot tree with tips colored by facility
#Read in tree
st147_tree <- read.tree('class18/st147.tree')

#Get vector of facilities for tips and nodes of tree
facils_tip <- c(facils[st147_tree$tip.label], 
              rep(NA, Nnode(st147_tree)))

#Plot tree with ggtree
ggtree(st147_tree) + 
  geom_tippoint(aes(col = facils_tip)) + 
  scale_color_discrete() +
  labs(col = 'Facility') 
```


```
##Extract phylogenetic clusters containing isolates from each facility
##and plot size distribution
#Get clusters with regentrans
clusters <- get_clusters(st147_tree_subset,facils[st147_tree_subset$tip.label], pureness = 1, pt = pts[st147_tree_subset$tip.label])

#Get pure subtree info for plotting
pure_subtree_info <- clusters$pure_subtree_info

#Plot size distribution of clusters
ggplot(data = pure_subtree_info, aes(x = loc, y = subtr_size, color = loc)) + 
  geom_jitter(position = position_jitter(width = 0.2, height = 0.1), alpha = 0.5) + 
  scale_color_discrete() +
  labs(y = "Number of isolates in \nphylogenetic cluster", x = "", color = 'Facility') +
  coord_flip()
```

Look for inter-facility transmission as evidenced by closely related isolates shared between facilities
-------------------------------------------------------------------------------------------------------

```
##Get number of closely related pairs between facilities and
##plot heatmap
#Subset to closely related pairs
pair_type_subset <- subset(pair_types, pairwise_dist < 10)

#Initialize matrix to hold facility pair counts
facil_pair_count = matrix(0, 
                          ncol = length(unique(facils)), 
                          nrow = length(unique(facils)),
                          dimnames = list(unique(facils), unique(facils) ))

#Add each inter-facility pair to appropriate facility pair
for(p in 1:nrow(pair_type_subset))
{
  
  facil_pair_count[pair_type_subset$loc1[p], pair_type_subset$loc2[p]] <- facil_pair_count[pair_type_subset$loc1[p], pair_type_subset$loc2[p]] + 1;
  facil_pair_count[pair_type_subset$loc2[p], pair_type_subset$loc1[p]] <- facil_pair_count[pair_type_subset$loc2[p], pair_type_subset$loc1[p]] + 1;
  
}

#Plot heatmap
pheatmap(log2(facil_pair_count+1), 
         cluster_rows = F, 
         cluster_cols = F)
```



