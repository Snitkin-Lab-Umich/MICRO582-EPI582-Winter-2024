Class 19 – Regional endemic spread
====================================

Goals
----
- Learn how to generate descriptive figures to characterize endemic spread across multiple healthcare facilities
- Learn how to interpret descriptive figures to understand where transmission is occuring

Setup
-----
We are going to be working in RStudio again today. Take the following steps to get ready for the lab:

1. Start up your epid582 Rproject and create a new directory in it called class19 to hold data we will be analyzing today. 
2. Go on to Great Lakes and copy over the class 19 files to your working directory
3. Use cyberduck to bring the files down to the class19 directory you created on your own computer

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


```
head(metadata)
class(aln)
dists[1:5, 1:5]
class(tr)
head(pt_trans_df)
```


```
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


```
#Get facility and patient named vectors
facils <- structure(metadata$facility, names = metadata$isolate_id)
pts <- structure(metadata$patient_id, names = metadata$isolate_id)

#Get pair types data frame from regentrans
pair_types <- get_pair_types(dists = dists, locs = facils, pt = pts)

#Plot histogram of intra- versus inter-facility pairwise distances
ggplot(data = pair_types, aes(x= pairwise_dist, fill = pair_type)) +
  geom_histogram(position = "identity", alpha = 0.4, bins = 50) + 
  labs(x = "Pairwise SNV distance", y = "Count", fill = "") + 
  xlim(-1,50)
```

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


Look for inter-facility transmission as evidenced by closely related isolates shared between facilities
-------------------------------------------------------------------------------------------------------

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


Compare genetic linkages between facilities to patient transfer network
-----------------------------------------------------------------------


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

