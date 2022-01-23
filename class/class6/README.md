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


Before we submit the job, run this command to make sure that prokka is setup properly in your environment.

```
prokka -setupdb
```

Now add these line at the end of the slurm script.

```
echo "Prokka results will be saved in $SLURM_WORKING_DIR/SRR5244781_prokka"
mkdir SRR5244781_prokka 
prokka -kingdom Bacteria -outdir SRR5244781_prokka -force -prefix SRR5244781_contigs_ordered SRR5244781_contigs_ordered.fasta

```

Next, let's explore some of the output files using our ever growing bag of Unix tricks :). First, lets take a look at the overall summary file that Prokka creates which indicates how many of each type of genomic feature was identified:

```
less SRR5244781_contigs_ordered.txt
```

You'll notice that this file is actually short, so instead of viewing in less it might be easier to dump the contents to the screen. You can do this with the 'cat' command:

```
cat SRR5244781_contigs_ordered.txt
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
  cut -f 3 SRR5244781_contigs_ordered.gff| grep $feat | wc -l; 

done

```
</details>

One more quick exercise - apply a Unix command to the appropriate fasta file to count the number of coding sequences and verify that it matches up with your loop.

<details>
  <summary>Solution</summary>

```
grep ">" SRR5244781_contigs_ordered.faa | wc -l

```
</details>



Functional annotation using eggnog
----------------------------------

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
