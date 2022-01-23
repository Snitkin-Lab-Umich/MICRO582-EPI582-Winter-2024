Class 6 – Genome annotation
===========================

Goal
----
- We will annotate the assembled genome from Class 5 using PROKKA
- We will also add functional annotations using eggnog to generate some rich annotatons for our genome.
- Then we will explore the annotated files to learn more about the assembled genome.

Genome annotation using PROKKA
------------------------------

Our genome asembly consists of long nucleotide sequences called contigs. However, having ~5Mb of sequence isn't super useful on it's own! What makes a genome informative are annotations, such that we can gain insight into the functions encoded in a genome and interpret the impact of genetic variation among a set of genomes. Over the past ~20 years there has been a great deal of bioinformatics research into how to identify functional elements in genomes, and in turn how to generate hypotheses regarding the molecular functions they perform and the biological processes in which they participate. Nowadays, there are pipelines available that combine different sets of tools in order to produce a fully annotated genome. For bacteria, the most commonly used tool is called Prokka. Prokka takes as input a genome assembly and performs the following functions:

1. Applies a tool called [Prodigal](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-11-119) to identify putative protein coding sequences.
2. Applies a tool called [RNAmmer](https://academic.oup.com/nar/article-abstract/35/9/3100/2401119) to identify rRNA encoding genes
3. Applies a tool called [Aragorn](https://academic.oup.com/nar/article-abstract/32/1/11/1194008) to identify tRNA encoding genes
4. Applies a tool called [Infernal](https://academic.oup.com/bioinformatics/article/29/22/2933/316439?login=true) to identify small RNA encoding genes

After applying these tools to identify functional elements, it then compares them to databases  of curated sequences (e.g. using BLAST) and transfers annotation. It is important to note that these annotations should be considered as bioinformatically generated hypotheses, with the confidence being associated with how closely related the sequence in the database is to the sequence you are annotating. 

Finally, the results of all of these annotations are stored in a series of standard genome annotation files, including:
1. .ffn - A fasta formated file of the nucleotide sequences of annotated features
2. .faa - A fasta formated file of the amino acid sequences of annotated coding regions
3. .gff - A gff formatted file of genome annotations
4. .gbk - A genbank formatted file of genome annotations and sequences
5. .tsv - A tab-separated value formatted file of genome annotations

With that background, let's try it on for ourselves! We are going to run Prokka on a Staphylococcus aureus genome that we downloaded the assembly for. Since Prokka annotation is a time intensive run, we will submit an annotation job and go over the results later at the end of this session. 

Lets add Prokka to our environment and check if we can invoke it.

Add this line at the end of your .bashrc file and source it.

```
export PATH=$PATH:/scratch/epid582w22_class_root/epid582w22_class/shared_data/bin/prokka/bin/
```

Let load some perl-modules that prokka requires and invoke prokka's help menu.

```
module load perl-modules

prokka -h
```

Before we submit the job, run this command to make sure that prokka is setup properly in your environment. - (we dont need to run this. when everyone tries to run this at once, it throws some error. Avoid setting it up because its already been set up by us)

```
prokka -setupdb
```

Lets copy over class6 data to your class working directory.

```
wd

cp -r /scratch/epid582w22_class_root/epid582w22_class/shared_data/data/class6 ./

cd class6
```

Now add these line at the end of the slurm script - .

```
echo "Prokka results will be saved in $SLURM_WORKING_DIR/SRR5244781_prokka"
mkdir SRR5244781_prokka 
prokka -kingdom Bacteria -outdir SRR5244781_prokka -force -prefix SRR5244781_contigs SRR5244781_contigs.fasta

```

Next, let's explore some of the output files using our ever growing bag of Unix tricks :). First, lets take a look at the overall summary file that Prokka creates which indicates how many of each type of genomic feature was identified:

```
cd SRR5244781_prokka

less SRR5244781_contigs.txt
```

You'll notice that this file is actually short, so instead of viewing in less it might be easier to dump the contents to the screen. You can do this with the 'cat' command:

```
cat SRR5244781_contigs.txt
```

Next, as an exercise, see if you can write a for loop that goes through the gff file created by Prokka and counts the number of occurences of each type of feature.

<details>
  <summary>Solution</summary>

```
for feat in CDS repeat_region rRNA tmRNA tRNA; 
do 
  #PRINT OUT THE NAME OF THE FEATURE
  echo $feat;
  
  #COUNT THE NUMBER OF OCCURENCES OF THE FEATURE IN THE THIRD COLUMN OF THE gff
  cut -f 3 SRR5244781_contigs.gff| grep $feat | wc -l; 

done

```
</details>

One more quick exercise - apply a Unix command to the appropriate fasta file to count the number of coding sequences and verify that it matches up with your loop.

<details>
  <summary>Solution</summary>

```
  
grep ">" SRR5244781_contigs.faa | wc -l
  
```
</details>



Functional annotation using eggnog
----------------------------------

Prokka provides you with some basic annotations (e.g. putative function of protein coding genes). However, it is often valuable to have richer annotations that place genes into different functional categories and/or pathway assignments such that:

1. You can gain a quick overview of how a genome is partitioned into different functional classes
2. You can compare genomes holistically to get a sense of how they are functionally different (e.g. nutrient sources they can utilize/environments they can grow in)
3. When evaluating the differences between pairs of genomes or sets of genomes (e.g. differences in gene content, gene expression, etc.), you can get a quick sense of the overall functional differences, which can lead to more focused and data-driven hypotheses

To get richer annotation of our genome we will apply a tool called Eggnog mapper. Eggnog mapper leverages a database of curated sequences (the Eggnog database), and applies an algorithm to map inputted genes to the most likely evolutionary counterpart (i.e. it's [ortholog](https://www.nature.com/articles/nrg3456)). There is a wide array of annotations that are provided, but two of special importance are [Gene Ontology](https://www.nature.com/articles/nrg3456) (GO) and [KEGG](https://www.genome.jp/kegg/) annotation. GO is an ontology scheme that assigned each gene to a biological process, molecular function and cellular component. One powerful aspect of GO is that it is hierarchical, such that you can categorize the role of genes at different levels (e.g. metabolism -> central carbon metabolism -> glycolysis). KEGG has a multitude of useful databases, but the most commonly used are it's pathway maps, which group genes into biochemical pathways and signalling cascades. 

Set up the Eggnog database directory so that Eggnog mapper knows where to look for the Eggnog Diamond database

```
export EGGNOG_DATA_DIR=/scratch/epid582w22_class_root/epid582w22_class/shared_data/database/eggnog
```

We already downloaded Eggnog Diamond database into this directory 

```
/scratch/epid582w22_class_root/epid582w22_class/shared_data/database/eggnog/
```

The command that was used to download the database was

```
download_eggnog_data.py
```

Open annotate.sbat file using nano and add commands for eggnog.

```
/scratch/epid582w22_class_root/epid582w22_class/shared_data/bin/eggnog-mapper/emapper.py -i SRR5244781_prokka/SRR5244781_contigs_ordered.faa --output SRR5244781_eggnog -m diamond --override --itype proteins
```

Submit the job using sbatch

```
sbatch annotate.sbat
```

The primary output file created by Eggnog mapper is a tab-delimited file providing different annotations for each gene (All_features_annotations.emapper.annotations). If you examine the file using less you will notice that the first three lines are comments, and the fourth line is a set of column headers. In order to explore specific annotations with Unix commands it would be super useful to know which annotations are on which columns. To do this you could tediously count across, but you should know by now that we don't like to do things manually :). So, let's check out a cool Unix trick to get what we want! There are two new concepts here. The first is the use of the 'tr' command, which we will use to "traslate" the tabs in the file to carriage returns. The second is the use of 'cat' with the -n flag, which provides line numbers to the output. To pull out the line of interest there are a variety of Unix commands we could employ, but to avoid introducing too many new concepts, let's just go with a grep solution:

```
grep "^#query" All_features_annotations.emapper.annotations | tr "\t" "\n" | cat -n
```

Next, let's get a sense for how the S. aureus genome is distributed among the different COG functional categories. From the above command you can see that the 21st column is COG functional categories. If you check out the file we provide 'COG_functional_categories.txt', you can see that each functional category is represented by a single letter code, and you can also see what the different categories are. Let's now pipe together some Unix commands we have used before to see all the unique COG categories in the Eggnog annotation file. We are also going to use one new flag (-c with uniq), which will tell us how many times each category occurs.

```
grep -v "#" All_features_annotations.emapper.annotations | cut -f 21 | sort | uniq -c 
```

One thing that's annoying about the above command is that it's in somewhat random order, so it's not easy to see which are the most common COG categories. Lucky for us - Unix can fix that for us! Let's add one more pipe to the 'sort' command, with flags to indicate which column to sort by (-k) and that we'd like to sort by numeric (-n).

```
grep -v "#" All_features_annotations.emapper.annotations | cut -f 21 | sort | uniq -c | sort -n -k 1
```

Now we can see how often annotations are observed in the Eggnog file, but one thing that would make this even easier to interpret is if we were also printing out the description of the COG code, so we wouldn't have to remember what they stand for. Essentially, what we want to do is loop through each COG category, print out the name in the lookup file and then count the number of times it occurs in the Eggnog output. How to do something so fancy though? Well, you already have all the tools to accomplish this! To make it more exciting and useful we are going to put these commands into a shell script that we could then use with any Eggnog mapper output file. 

We have started the code, but you have to add two commands in the for loop to make it work :). Open 'COG_code_counter.sh' in nano and add the appropriate commands below the comments in the for loop.

```
#GO THROUGH EACH COG 1-LETTER CODE AND PRINT OUT NAME AND COUNT FROM EGG NOG OUTPUT FILE
for COG_code in `cut -f 1 $2`
do

        #GET THE NAME OF THE COG CATEGORY FROM COG_functional_categories.txt


        #COUNT THE NUMBER OF OCCURENCES IN COLUMN 21 OF THE E-MAPPER DATA

done
```

<details>
  <summary>Solution</summary>

```
#GO THROUGH EACH COG 1-LETTER CODE AND PRINT OUT NAME AND COUNT FROM EGG NOG OUTPUT FILE
for COG_code in `cut -f 1 $2`
do

        #GET THE NAME OF THE COG CATEGORY FROM COG_functional_categories.txt
        grep ^$COG_code $2;

        #COUNT THE NUMBER OF OCCURENCES IN COLUMN 21 OF THE E-MAPPER DATA
        cut -f 21 $1 | grep $COG_code | wc -l

done
```
</details>
