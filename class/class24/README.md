Class 25 – SARS-CoV-2 dated phylogenetic analysis
=================================================
Goals
----
- Learn about the impact that variant filtering can have on data analysis
- Learn how to use the treetime tool for dated phylogenetic inference
- Learn about ancestral reconstruction of location states
- Learn how to plot time-scaled trees in R

Setup
-----
We are going to be working in RStudio again today. Take the following steps to get ready for the lab:

1. Start up your epid582 Rproject and create a new directory in it called class25 to hold data we will be analyzing today. 
2. Go on to Great Lakes and copy over the class 25 files to your working directory
3. Use cyberduck to bring the files down to the class25 directory you created on your own computer

Background on lab and data set
------------------------------
Today we will be working with some of the same data from last week, which was comprised of samples collected by Rush University Medical Center in March and April 2020. Importantly, Rush was the primary testing/processing location for Chicago during this time period, so the samples actually provide a reasonable picture of Chicago early in the pandemic. Last week we were focused on understanding putative transmission within Rush between patients and healthcare workers. Today, we are going to try and put all of the isolates from Chicago in the context of the larger pandemic to understand where and when SARS-CoV-2 entered Chicago from and how many times. To this end we will supplement our Rush collection with publically available genomes deposited in [GISAID](https://www.gisaid.org/). In order to alleviate some of the bias in publically available genomes due to uneven sequencing, we downsampled genomes from certain states in the U.S. (e.g. NY, MA) and from certain countries (e.g. UK). If your interested, I can provide more specifics on our downsampling procedure, but in general we tried to sample evenly over time, given the constraint that we wanted to downsample proportionally from certain locations.

Impact of variant filtering
---------------------------
To perform our dated phytlogenetic analysis we are going to use a tool called [treetime](https://github.com/neherlab/treetime). Treetime has several functionalities, including estimation of a time-scaled phylogeny and estimation of ancestral locations (e.g. origin of unobserved ancestral SARS-CoV-2 strains). 

As a first step before creating our time-scaled tree it is good practice to verify that your data has "temporal signal". Temporal signal means that there is statistical support for genetic variation accumulating at a contant rate over time. This is important, because we will be using the amount of genetic variation between isolates to estimate the time since their common ancestor. So, if there is no temporal signal, then there is good reason to be skeptical of downstream inferences, because the the underlying hypothesis that variation accumulates according to some evolutionary rate is not supported. 

As was discussed last class and demonstrated in the lab, it is critical to perform proper QC and variant filtering with sequence data. Last time we applied this filtering to sequence data that we generated. Here, we are going to see the impact of not applying variant filtering on sequences that are present in public databases.

We have run the following treetime commands to assess evidence of a temporal signal in SARS-CoV-2 genome alignments that did and did not undergo the variant masking procedure described last time. Note that in both data sets the genomes from were properly masked, so differences are simply a reflection of poor sequence quality in public databases.

```
##Clock test
#Unmasked
treetime clock --tree ML_tree_unmasked.tree --dates unmasked_tree_time_metadata.csv --name-column SeqID --date-column Date_float --aln unmasked_covid_sequences.fa --outdir clock_results_unmasked --clock-filter 3


#Masked
treetime clock --tree ML_tree_masked.tree --dates masked_tree_time_metadata.csv --name-column SeqID --date-column Date_float --aln masked_covid_sequences.fa --outdir clock_results_masked --clock-filter 3
```

Among the outputs of these commands is a pdf plot of root-to-tip distance versus time (masked_root_to_tip_regression.pdf and unmasked_root_to_tip_regression.pdf). The root-to-tip distance is the phylogenetic distance from a tip (i.e. an isolate) to the root of the tree, and time is the date of isolation. If there is a temporal signal we expect isolates with later dates to be farther from the root (i.e. more time for variation to accumulate), such that there is a positive trend line in the plot. In addition, we can learn two more things from this plot:

1) The slope of the line fit between phylogenetic distance and time is the evolutionary rate (i.e. distance/time). 
2) The x-intercept (i.e. zero distance from the root) is the inferred date of the most recent common ancestor of your tree (i.e. root date)

Comparing the masked (masked_clock_run) and unmasked (unmasked_clock_run) data we see the following differences:

1) The correlation coefficient for the two plots is very different (R2 = 0.51 for masked and R2 = 0.06 for unmasked). This suggests a stronger temporal signal in the masked data.
2) The inferred root-date of the two data sets is also different, with the unmasked having a predicted root-date of May/June 2019, and the masked data having a predicted root-date of December 2019.


Ancestral reconstruction of location
------------------------------------
The second function we are going to explore withe timetree is performing ancestral reconstruction of location on our time-scaled tree. This essentially implements a maximum likelihood algorithm to infer the most likely ancestral location of unobserved anscestral sequences, given the known locations of sequences in our collection. We can use this ancestral reconstruction to make inferences regarding inter-region transmission, and with respect to our data, where SARS-CoV-2 in Chicago came from.

Below are the commands we ran to generate our time-scaled tree and then perform ancestral reconstruciton.

```
#Create time-scaled tree
treetime --tree ML_tree_masked.tree --dates masked_tree_time_metadata.csv --name-column SeqID --date-column Date_float --aln masked_covid_sequences.fa --outdir tree_results_masked --clock-filter 3  --reconstruct-tip-states

#Perform ancestral reconstruction on time-scaled tree
treetime mugration --tree tree_results_masked/timetree.nexus --states masked_tree_time_metadata.csv --outdir continentAR_masked/  --attribute ContAR
```


Visualizing time-scaled trees in R
----------------------------------
Lastly, we are going to read in our time-scaled ancestrally reconstructed trees into R and visualize them with ggtree.

```
#Read in tree
contAR_tree = read.beast('continent_AR_tree_subset.tree')


#Explore the tree data
contAR_tree_data = as_tibble(contAR_tree)
contAR_phylo = as.phylo(contAR_tree)


#Plot ancestrally reconstructed tree
ggtree(contAR_tree, root.position = 2019.91, aes(color=ContAR), alpha = 0.35) + 
  theme_tree2() + 
  geom_tippoint(aes(color=ContAR), size=0.75, alpha=.75) + 
  scale_color_manual(values=brewer.pal(8, 'Set2')) +
    theme(legend.position="right")
```




