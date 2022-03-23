Class 19 – Regional endemic spread
====================================

Goals
----
- Learn how to generate descriptive figures to characterize endemic spread across multiple healthcare facilities
- Learn how to interpret descriptive figures to understand where transmission is occuring

Setup
-----
We are going to be working in RStudio again today. Take the following steps to get ready for the lab:

1. No need to download any files today - we will be using the data that comes with regentrans.

Once in R, load the following libraries

```
library(regentrans)
library(ape)
library(ggplot2)
library(tibble)
library(tidyverse)
library(ggtree)
library(pheatmap)
```

Background on the data
----------------------
For today's lab we will be working on a sample and data collection comprising carbapenem-resistant Klebsiella pneumoniae (CRKP) collected from 21 long-term acute care hospitals (LTACHs) in the United States. The majority of isolates came from 11 LTACHs in LA County, where CRKP is highly prevalent due to the circulation of epidemic lineage ST258. In today's lab we are going to use regentrans to replicate some of the analyses from our [publication](https://pubmed.ncbi.nlm.nih.gov/31451495/) on this data, where we showed how genomic analysis could inform where regional transmission is occurring, and the pathways of transmission between regional LTACHs. 

The data from this manuscript comes as the default dataset in regentrans, so just loading the library gives you access to the variables below!

Also, check out the great [vignette](https://snitkin-lab-umich.github.io/regentrans/articles/Introduction.html) that Sophie and Zena created showing you different types of analayses/visualizations that regentrans can enable. We will just look at a couple today.

```
#Preloaded dataset that comes with regentrans

head(metadata) #meta-data of study isolates
class(aln) #variant alignment of ST258 genomes
dists[1:5, 1:5] #distance matrix constructed from variant alignment
class(tr) #maximum likelihood tree constructed from variant alignment
head(pt_trans_df) #counts of patient transfers between pairs of healthcare facilities
```

To start, let's plot a tree of the isolates with facility overlaid. With a dataset this large, it's hard to see much just from this type of visualization :)
```
#Get facility and patient named vectors
facils <- structure(metadata$facility, names = metadata$isolate_id)
pts <- structure(metadata$patient_id, names = metadata$isolate_id)

#Plot tree with facilities overlaid
facils_tip <- c(facils[tr$tip.label],
                rep(NA, Nnode(tr)))

ggtree(tr) +
  geom_tippoint(aes(col = facils_tip)) +
  scale_color_discrete() +
  labs(col = "Facility")
```

Calculate pairwise genetic distances and look for evidence of transmission within facilities
--------------------------------------------------------------------------------------------
As we did in the previous class for our regional outbreak, we are going to look at pairwise SNV distances among isolates from the same and different facilities to:
1. Assess whether we are detecting recent transmission in our data set as evidenced by closely related pairs
2. If we see evidence of a appropraite SNV threshold for recent transmission, as evidenced by degradation for the enrichment in epidemiologic association with increasing SNV thresholds


To start, let's plot our histograms colored by inta- and inter-facility pairs.
```
#Get pair types data frame from regentrans
pair_types <- get_pair_types(dists = dists, locs = facils, pt = pts)

#Plot histogram of intra- versus inter-facility pairwise distances
ggplot(data = pair_types, aes(x= pairwise_dist, fill = pair_type)) +
  geom_histogram(position = "identity", alpha = 0.4, bins = 50) + 
  labs(x = "Pairwise SNV distance", y = "Count", fill = "") + 
  xlim(-1,50)
```

As with last time, it does appear that there is an enrichment among isolates from the same facility at the smallest SNV distances. So, let's look at how this enrichment tails off, and whether there is a reasonable SNV threshold for recent transmission. 

```
#Get fraction of intra-facility pairs at different SNV cutoffs
frac_intra <- get_frac_intra(pair_types = pair_types)

#Plot barplot
ggplot(data = frac_intra, aes(x= pairwise_dist, y = frac_intra)) +
  geom_bar(stat = "identity", alpha = 0.5) +
  labs(x = "Pairwise SNV distance", y = "Fraction of intra-facility pairs") +
  ylim(0,1) + 
  xlim(0,50)
```
From this it looks like 8 or 10 SNVs could be a good threshold. This is actually somewhat reassuring as the evolutionary rate of ST258 is 4 SNVs per genome per year. So, since our study was a year long, it is reasonable to expect that isolates linked by recent transmission could be within this range of genetic distance, which could accumulate over the course of a year.

Look for inter-facility transmission as evidenced by closely related isolates shared between facilities
-------------------------------------------------------------------------------------------------------

Next, we are going to use our SNV threshold of 10 SNVs to look at the number of putative transmission linkages identified within and between each pair of facilities, and plot a heatmap of the results.

```
#Calculate pairwise linkages among facilities
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

You can see that there is variation in the amount of intra-facility transmission, as well as the amount of connection between different pairs of facilities.

Compare genetic linkages between facilities to patient transfer network
-----------------------------------------------------------------------
Finally, we are going to evaluate how the amount of putative transmission between facilities as inferred from genomic ananlysis relates to connectivity between facilities as determined by patient transfer data. 

For patient transfer data we are going to use a function in regentrans to calculate the patient flow between each pair of facilities. Essentially, this takes into account all the direct and indirect pathways between each pair of facilities, and sums them up to estimate overall connectivity between pairs of facilities.

For genomic data we will use two metrics to quantify density of transmission. The first is the previously calculated number of isolate pairs below our SNV threshold. The second is a metric called fsp (which is computed by regentrans). Fsp is a population genetic metric that quantifies the amount of intermixing between two populations, or in our case between two facilities. In essence, this metric quantifies the amount of shared genetic variation among isolates from pairs of facilities, and in this way gets at inter-facility transmission without having to impose an SNV threshold.


```
#Compare pairwise linkages to patient transfer network
pt_trans <- get_patient_flow(pt_trans_df = pt_trans_df)
head(pt_trans)

pt_trans$num_pairs_lt10 <- facil_pair_count[cbind(pt_trans$loc1, pt_trans$loc2)]
pt_trans$fsp <- fsp[cbind(pt_trans$loc1, pt_trans$loc2)]

ggplot(data = pt_trans, aes(x=sum_pt_trans_metric,y=num_pairs_lt10)) +
  geom_point() + geom_smooth(method='lm') + scale_x_log10() +
  labs(x = 'Patient flow', y = '# closely related\npairs (≤ 10 SNVs)') 


ggplot(data = pt_trans, aes(x=sum_pt_trans_metric,y=fsp)) +
  geom_point() + geom_smooth(method='lm') + scale_x_log10() +
  labs(x = 'Patient flow', y = 'fsp') 
```

