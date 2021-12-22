Class 6 – Genome annotation
===========================

Goal
----
- We will annotate the assembled genome from Class 5 using PROKKA
- We will also add functional annotations using eggnog to generate some rich annotatons for our genome.
- Then we will explore the annotated files to learn more about the assembled genome.

Annotate genome with PROKKA and eggnogg
---------------------------------------

Since Prokka annotation is a time intensive run, we will submit an annotation job and go over the results later at the end of this session. 


Before we submit the job, run this command to make sure that prokka is setup properly in your environment.

```
prokka -setupdb
```

In your day2am directory, you will find a prokka.sbat script. Open this file using nano and change the EMAIL_ADDRESS to your email address.

```
nano prokka.sbat

```

Now add these line at the end of the slurm script.

```
echo "The job will run from - $SLURM_WORKING_DIR"
echo "Prokka results will be saved in $SLURM_WORKING_DIR/SRR5244781_prokka"
mkdir SRR5244781_prokka 
prokka -kingdom Bacteria -outdir SRR5244781_prokka -force -prefix SRR5244781_contigs_ordered SRR5244781_contigs_ordered.fasta

```

Submit the job using sbatch

```
sbatch prokka.sbat
```

