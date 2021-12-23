Class 6 – Genome annotation
===========================

Goal
----
- We will annotate the assembled genome from Class 5 using PROKKA
- We will also add functional annotations using eggnog to generate some rich annotatons for our genome.
- Then we will explore the annotated files to learn more about the assembled genome.

Genome annotation using PROKKA
------------------------------

Since Prokka annotation is a time intensive run, we will submit an annotation job and go over the results later at the end of this session. 


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

Eggnog requires the latest version of diamond and the version we installed with Conda package manager doesn't download the latest version. To get a workaround this, we downloaded diamond executable to this directory `/scratch/epid582w22_class_root/epid582w22_class/shared_data/bin/diamond` from diamond's github [website](https://github.com/bbuchfink/diamond/wiki) 

Lets copy this executable to your conda's bin directory. Lets find out where your conda bin folder is located. To find out where diamond is located, run the below commands

```
which diamond
```

My conda diamond version is located here - `~/.conda/envs/MICRO582/bin/diamond`. Lets move the latest version to this path. (Offcourse your location might be different. So you should copy `/scratch/epid582w22_class_root/epid582w22_class/shared_data/bin/diamond` to your location)

```
cp /scratch/epid582w22_class_root/epid582w22_class/shared_data/bin/diamond ~/.conda/envs/MICRO582/bin/diamond
```


Open annotate.sbat file using nano and add commands for eggnog.

```
/scratch/epid582w22_class_root/epid582w22_class/shared_data/bin/eggnog-mapper/emapper.py -i SRR5244781_prokka/SRR5244781_contigs_ordered.faa --output SRR5244781_eggnog -m diamond --override --itype proteins
```

Submit the job using sbatch

```
sbatch annotate.sbat
```