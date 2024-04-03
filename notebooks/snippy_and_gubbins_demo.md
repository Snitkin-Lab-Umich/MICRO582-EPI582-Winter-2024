# Tutorial on running Snippy, Gubbins, AMRFinderPlus and MLST tools on entire dataset.

This is a demo showing how you can apply various bioinformatics tools on multiple samples/entire dataset. You can closely follow these steps and adapt the below commands for your final project dataset. 

For the purpose of this demo, we will use dataset from Assignment 2 which is located here:

```/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/```



#### 1. Run Snippy and Gubbins on CRKP dataset

For more in-depth information about various parameters, please follow official Snippy and Gubbins documentation.

Link to Snippy Github: https://github.com/tseemann/snippy

Link to Gubbins Github: https://github.com/nickjcroucher/gubbins

Since, Gubbins is run on the final output of Snippy, we will run Snippy and Gubbins in the same folder.
 
- We will create a new directory to save Snippy and Gubbins results under shared_data - ```shared_data/data/snippy_and_gubbins_demo```


```bash
cd /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/
```


```bash
mkdir snippy_and_gubbins_demo
```


```bash
cd snippy_and_gubbins_demo
```

- Prepare multiple sample list for Snippy

The goal of this demo is to show how you can ask Snippy to perform variant calling on multiple samples in one go rather than running it individually on each sample. 

Snippy package comes with a helper script called ```snippy-multi``` that takes a tab seperated list of Sample names and directory path to its fastq sequences, generates a bash script containing Snippy commands for each samples which can then be submitted as a slurm job.

We will use unix for loop, cut and sed command to create a tab seperated file containing three columns seperated by a tab: 

```SAMPLENAME1    PATH-TO-SAMPLENAME1_FORWARD_READ    PATH-TO-SAMPLENAME1_REVERSE_READ)```



```bash
#Loop through each file of forward reads, and print genome name and F/R read files for snippy batch
for r1 in /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/*_1.fastq.gz;
do 
    #Get name of genome from forward reads
    isolate=`echo $r1 | cut -d'/' -f9 | sed 's/_1.fastq.gz//g'`;
    #Get reverse reads corresponding to current forward reads
    r2=`echo $r1 | sed 's/_1.fastq/_2.fastq/g'`;
    #Print out genome, forward reads and reverse reads, separated by tabs
    printf "$isolate\t$r1\t$r2\n";
done > input.tab
```

***Lets check the input.tab file we just created:***


```bash
head input.tab
```

    SRR6204326	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204326_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204326_2.fastq.gz
    SRR6204327	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204327_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204327_2.fastq.gz
    SRR6204328	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204328_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204328_2.fastq.gz
    SRR6204329	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204329_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204329_2.fastq.gz
    SRR6204330	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204330_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204330_2.fastq.gz
    SRR6204332	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204332_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204332_2.fastq.gz
    SRR6204334	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204334_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204334_2.fastq.gz
    SRR6204335	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204335_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204335_2.fastq.gz
    SRR6204336	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204336_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204336_2.fastq.gz
    SRR6204337	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204337_1.fastq.gz	/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204337_2.fastq.gz


***The first column contains samplenames that Snippy will use as a prefix to create output foldername***

- Run snippy-multi on input sample list

- Snippy is available as a module on Great Lakes. Lets load Snippy and check if all the dependencies looks good. 


```bash
# Load these modules
module load Bioinformatics snippy/4.6.0
```

    
        Provides Bioinformatics software.
        For more information please use:
    
            $ module help Bioinformatics
    
    



```bash
snippy --check
```

    [15:36:56] This is snippy 4.5.1
    [15:36:56] Written by Torsten Seemann
    [15:36:56] Obtained from https://github.com/tseemann/snippy
    [15:36:56] Detected operating system: linux
    [15:36:56] Enabling bundled linux tools.
    [15:36:56] Found bwa - /opt/conda/bin/bwa
    [15:36:56] Found bcftools - /opt/conda/bin/bcftools
    [15:36:56] Found samtools - /opt/conda/bin/samtools
    [15:36:56] Found java - /opt/conda/bin/java
    [15:36:56] Found snpEff - /opt/conda/bin/snpEff
    [15:36:56] Found samclip - /opt/conda/bin/samclip
    [15:36:56] Found seqtk - /opt/conda/bin/seqtk
    [15:36:56] Found parallel - /opt/conda/bin/parallel
    [15:36:56] Found freebayes - /opt/conda/bin/freebayes
    [15:36:56] Found freebayes-parallel - /opt/conda/bin/freebayes-parallel
    [15:36:56] Found fasta_generate_regions.py - /opt/conda/bin/fasta_generate_regions.py
    [15:36:56] Found vcfstreamsort - /opt/conda/bin/vcfstreamsort
    [15:36:56] Found vcfuniq - /opt/conda/bin/vcfuniq
    [15:36:56] Found vcffirstheader - /opt/conda/bin/vcffirstheader
    [15:36:56] Found gzip - /bin/gzip
    [15:36:56] Found vt - /opt/conda/bin/vt
    [15:36:56] Found snippy-vcf_to_tab - /opt/conda/bin/snippy-vcf_to_tab
    [15:36:56] Found snippy-vcf_report - /opt/conda/bin/snippy-vcf_report
    [15:36:56] Checking version: samtools --version is >= 1.7 - ok, have 1.10
    [15:36:56] Checking version: bcftools --version is >= 1.7 - ok, have 1.10
    [15:36:56] Checking version: freebayes --version is >= 1.1 - ok, have 1.3.2
    [15:36:56] Checking version: snpEff -version is >= 4.3 - ok, have 4.3
    [15:36:56] Checking version: bwa is >= 0.7.12 - ok, have 0.7.17
    [15:36:56] Dependences look good!


- Only proceed if the last line of the snippy check is "Dependences look good!"

We will run snippy-multi on commandline with input.tab as your input and path to a reference genome genbank file. 

Note: Make sure to use project specific reference genome while adapting this command for your Final project.


```bash
snippy-multi input.tab --ref /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/class9/KPNIH1.gbk --cpu 4 --force --report > runme.sh
```

    Reading: input.tab
    Generating output commands for 56 isolates
    Done.


Note: 

    - snippy-multi generates individual snippy commands for each sample as well as a single snippy-core command at the end. snippy-core basically aggregates results from individual samples and generates a core consensus alignment file. This consensus alignment file will then be used as an input for Gubbins.

    - If snippy-multi throws an error that it can't read a read file or a path, its propbably due to the format of your input.tab Check your for loop and regenerate the input.tab file. This file should contain three columns each seperated by a tab. 


```bash
head -n5 runme.sh
```

    snippy --outdir 'SRR6204326' --R1 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204326_1.fastq.gz' --R2 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204326_2.fastq.gz' --ref /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/class9/KPNIH1.gbk --cpu 4 --force --report
    snippy --outdir 'SRR6204327' --R1 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204327_1.fastq.gz' --R2 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204327_2.fastq.gz' --ref /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/class9/KPNIH1.gbk --cpu 4 --force --report
    snippy --outdir 'SRR6204328' --R1 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204328_1.fastq.gz' --R2 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204328_2.fastq.gz' --ref /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/class9/KPNIH1.gbk --cpu 4 --force --report
    snippy --outdir 'SRR6204329' --R1 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204329_1.fastq.gz' --R2 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204329_2.fastq.gz' --ref /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/class9/KPNIH1.gbk --cpu 4 --force --report
    snippy --outdir 'SRR6204330' --R1 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204330_1.fastq.gz' --R2 '/scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/outbreak_fastq/SRR6204330_2.fastq.gz' --ref /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/class9/KPNIH1.gbk --cpu 4 --force --report


***If you look at the last line of runme.sh, you would find a snippy-core command***


```bash
tail -n1 runme.sh
```

    snippy-core --ref 'SRR6204326/ref.fa' SRR6204326 SRR6204327 SRR6204328 SRR6204329 SRR6204330 SRR6204332 SRR6204334 SRR6204335 SRR6204336 SRR6204337 SRR6204339 SRR6204340 SRR6204341 SRR6204343 SRR6204345 SRR6204346 SRR6204347 SRR6204348 SRR6204349 SRR6204350 SRR6204351 SRR6204352 SRR6204354 SRR6204355 SRR6204358 SRR6204360 SRR6204361 SRR6204362 SRR6204363 SRR6204364 SRR6204365 SRR6204366 SRR6204367 SRR6204368 SRR6204369 SRR6204370 SRR6204373 SRR6204374 SRR6204375 SRR6204376 SRR6204377 SRR6204378 SRR6204381 SRR6204382 SRR6204383 SRR6204384 SRR6204385 SRR6204386 SRR6204387 SRR6204388 SRR6204389 SRR6204390 SRR6204391 SRR6204392 SRR6204393 SRR6204394


- Add this runme.sh bash script in a slurm sbat file.


```bash
touch snippy.sbat
```

**Substitute username with your umich uniqname and paste these lines to snippy.sbat file using nano:**

```
#!/bin/sh
# Job name
#SBATCH --job-name=Snippy
# User info
#SBATCH --mail-user=username@umich.edu
#SBATCH --mail-type=BEGIN,END,FAIL,REQUEUE
#SBATCH --export=ALL
#SBATCH --partition=standard
#SBATCH --account=epid582w24_class
# Number of cores, amount of memory, and walltime
#SBATCH --nodes=1 --ntasks=1 --cpus-per-task=4 --mem=20g --time=12:00:00

echo $SLURM_SUBMIT_DIR

bash runme.sh
```

Submit the job using sbatch


```bash
sbatch snippy.sbat
```

    Submitted batch job 5795798


Snippy will generate variant calling results for each of the samples along with other useful reports that we can analyze further based on our needs. 

Extension | Description
----------|--------------
.aln | A core SNP alignment in the `--aformat` format (default FASTA)
.full.aln | A whole genome SNP alignment (includes invariant sites)
.tab | Tab-separated columnar list of **core** SNP sites with alleles but NO annotations
.vcf | Multi-sample VCF file with genotype `GT` tags for all discovered alleles
.txt | Tab-separated columnar list of alignment/core-size statistics
.ref.fa | FASTA version/copy of the `--ref`
.self_mask.bed | BED file generated if `--mask auto` is used.


```bash
head core.txt
```

    ID	LENGTH	ALIGNED	UNALIGNED	VARIANT	HET	MASKED	LOWCOV
    SRR6204326	5766615	5461922	286542	208	2437	0	15714
    SRR6204327	5766615	5419126	292048	189	11792	0	43649
    SRR6204328	5766615	5312074	292778	168	7127	0	154636
    SRR6204329	5766615	5460187	286745	176	3617	0	16066
    SRR6204330	5766615	5463038	284579	173	2810	0	16188
    SRR6204332	5766615	5450687	286854	176	4767	0	24307
    SRR6204334	5766615	5397586	354903	164	1474	0	12652
    SRR6204335	5766615	5453872	287043	170	1345	0	24355
    SRR6204336	5766615	5441449	287387	170	10522	0	27257


- Run Gubbins on multi sequence alignemnt `core.full.aln`


```bash
# Unload any previously loaded modules.
module purge
# Load Gubbins
module load Bioinformatics gubbins/2.3.1
```

    
        Provides Bioinformatics software.
        For more information please use:
    
            $ module help Bioinformatics
    
    



```bash
# Check if gubbins was loaded into the environment
run_gubbins -h
```

    usage: run_gubbins [-h] [--outgroup OUTGROUP] [--starting_tree STARTING_TREE]
                       [--use_time_stamp] [--verbose] [--no_cleanup]
                       [--tree_builder TREE_BUILDER] [--iterations ITERATIONS]
                       [--min_snps MIN_SNPS]
                       [--filter_percentage FILTER_PERCENTAGE] [--prefix PREFIX]
                       [--threads THREADS] [--converge_method CONVERGE_METHOD]
                       [--version] [--min_window_size MIN_WINDOW_SIZE]
                       [--max_window_size MAX_WINDOW_SIZE]
                       [--raxml_model RAXML_MODEL] [--remove_identical_sequences]
                       alignment_filename
    
    Croucher N. J., Page A. J., Connor T. R., Delaney A. J., Keane J. A., Bentley
    S. D., Parkhill J., Harris S.R. "Rapid phylogenetic analysis of large samples
    of recombinant bacterial whole genome sequences using Gubbins". Nucleic Acids
    Res. 2015 Feb 18;43(3):e15. doi: 10.1093/nar/gku1196 .
    
    positional arguments:
      alignment_filename    Multifasta alignment file
    
    optional arguments:
      -h, --help            show this help message and exit
      --outgroup OUTGROUP, -o OUTGROUP
                            Outgroup name for rerooting. A list of comma separated
                            names can be used if they form a clade (default: None)
      --starting_tree STARTING_TREE, -s STARTING_TREE
                            Starting tree (default: None)
      --use_time_stamp, -u  Use a time stamp in file names (default: 0)
      --verbose, -v         Turn on debugging (default: 0)
      --no_cleanup, -n      Dont cleanup intermediate files (default: 0)
      --tree_builder TREE_BUILDER, -t TREE_BUILDER
                            Application to use for tree building
                            [raxml|fasttree|hybrid] (default: raxml)
      --iterations ITERATIONS, -i ITERATIONS
                            Maximum No. of iterations (default: 5)
      --min_snps MIN_SNPS, -m MIN_SNPS
                            Min SNPs to identify a recombination block (default:
                            3)
      --filter_percentage FILTER_PERCENTAGE, -f FILTER_PERCENTAGE
                            Filter out taxa with more than this percentage of gaps
                            (default: 25)
      --prefix PREFIX, -p PREFIX
                            Add a prefix to the final output filenames (default:
                            None)
      --threads THREADS, -c THREADS
                            Number of threads to run with RAXML, but only if a
                            PTHREADS version is available (default: 1)
      --converge_method CONVERGE_METHOD, -z CONVERGE_METHOD
                            Criteria to use to know when to halt iterations [weigh
                            ted_robinson_foulds|robinson_foulds|recombination]
                            (default: weighted_robinson_foulds)
      --version             show program's version number and exit
      --min_window_size MIN_WINDOW_SIZE, -a MIN_WINDOW_SIZE
                            Minimum window size (default: 100)
      --max_window_size MAX_WINDOW_SIZE, -b MAX_WINDOW_SIZE
                            Maximum window size (default: 10000)
      --raxml_model RAXML_MODEL, -r RAXML_MODEL
                            RAxML model [GTRGAMMA|GTRCAT] (default: GTRCAT)
      --remove_identical_sequences, -d
                            Remove identical sequences (default: 0)



```bash
touch gubbins.sbat
```

Copy and paste these lines to gubbins.sbat file using nano:

```
#!/bin/sh
# Job name
#SBATCH --job-name=Gubbins
# User info
#SBATCH --mail-user=username@umich.edu
#SBATCH --mail-type=BEGIN,END,FAIL,REQUEUE
#SBATCH --export=ALL
#SBATCH --partition=standard
#SBATCH --account=epid582w24_class
# Number of cores, amount of memory, and walltime
#SBATCH --nodes=1 --ntasks=1 --cpus-per-task=8 --mem=40g --time=12:00:00
#  Change to the directory you submitted from
cd $SLURM_SUBMIT_DIR
echo $SLURM_SUBMIT_DIR

run_gubbins --prefix crkp_core_full_aln --verbose /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/snippy_and_gubbins_demo/core.full.aln
```


```bash
sbatch gubbins.sbat
```

- Gubbins will create various output files with the prefix `crkp_core_full_aln`. 
- We will move these files to a new folder gubbins_results.


```bash
mkdir gubbins_results
```


```bash
mv crkp_core_full_aln.* gubbins_results/
```

#### 2. Run AMRFinderPlus on CRKP assemblies


```bash
cd /scratch/epid582w24_class_root/epid582w24_class/shared_data/data
```


```bash
mkdir AMRFinderPlus_demo
```


```bash
cd AMRFinderPlus_demo
```


```bash
conda activate class7
```

    (class7) 




```bash
# Check if amrfinder is loaded into the environment
amrfinder -h
```

    (class7) Identify AMR and virulence genes in proteins and/or contigs and print a report
    
    DOCUMENTATION
        See https://github.com/ncbi/amr/wiki for full documentation
    
    UPDATES
        Subscribe to the amrfinder-announce mailing list for database and software update notifications:
        https://www.ncbi.nlm.nih.gov/mailman/listinfo/amrfinder-announce
    
    USAGE:   amrfinder [--update] [--force_update] [--protein PROT_FASTA] [--nucleotide NUC_FASTA] [--gff GFF_FILE] [--annotation_format ANNOTATION_FORMAT] [--database DATABASE_DIR] [--database_version] [--ident_min MIN_IDENT] [--coverage_min MIN_COV] [--organism ORGANISM] [--list_organisms] [--translation_table TRANSLATION_TABLE] [--plus] [--report_common] [--report_all_equal] [--name NAME] [--print_node] [--mutation_all MUT_ALL_FILE] [--output OUTPUT_FILE] [--protein_output PROT_FASTA_OUT] [--nucleotide_output NUC_FASTA_OUT] [--nucleotide_flank5_output NUC_FLANK5_FASTA_OUT] [--nucleotide_flank5_size NUC_FLANK5_SIZE] [--blast_bin BLAST_DIR] [--hmmer_bin HMMER_DIR] [--quiet] [--pgap] [--gpipe_org] [--parm PARM] [--threads THREADS] [--debug] [--log LOG]
    HELP:    amrfinder --help or amrfinder -h
    VERSION: amrfinder --version
    
    NAMED PARAMETERS:
    -u, --update
        Update the AMRFinder database
    -U, --force_update
        Force updating the AMRFinder database
    -p PROT_FASTA, --protein PROT_FASTA
        Input protein FASTA file (can be gzipped)
    -n NUC_FASTA, --nucleotide NUC_FASTA
        Input nucleotide FASTA file (can be gzipped)
    -g GFF_FILE, --gff GFF_FILE
        GFF file for protein locations (can be gzipped). Protein id should be in the attribute 'Name=<id>' (9th field) of the rows with type 'CDS' or 'gene' (3rd field).
    -a ANNOTATION_FORMAT, --annotation_format ANNOTATION_FORMAT
        Type of GFF file: bakta, genbank, microscope, patric, pgap, prodigal, prokka, pseudomonasdb, rast, standard
        Default: genbank
    -d DATABASE_DIR, --database DATABASE_DIR
        Alternative directory with AMRFinder database. Default: $AMRFINDER_DB
    -V, --database_version
        Print database version
    -i MIN_IDENT, --ident_min MIN_IDENT
        Minimum proportion of identical amino acids in alignment for hit (0..1). -1 means use a curated threshold if it exists and 0.9 otherwise
        Default: -1
    -c MIN_COV, --coverage_min MIN_COV
        Minimum coverage of the reference protein (0..1)
        Default: 0.5
    -O ORGANISM, --organism ORGANISM
        Taxonomy group. To see all possible taxonomy groups use the --list_organisms flag
    -l, --list_organisms
        Print the list of all possible taxonomy groups for mutations identification and exit
    -t TRANSLATION_TABLE, --translation_table TRANSLATION_TABLE
        NCBI genetic code for translated BLAST
        Default: 11
    --plus
        Add the plus genes to the report
    --report_common
        Report proteins common to a taxonomy group
    --report_all_equal
        Report all equally-scoring BLAST and HMM matches
    --name NAME
        Text to be added as the first column "name" to all rows of the report, for example it can be an assembly name
    --print_node
        print hierarchy node (family)
    --mutation_all MUT_ALL_FILE
        File to report all mutations
    -o OUTPUT_FILE, --output OUTPUT_FILE
        Write output to OUTPUT_FILE instead of STDOUT
    --protein_output PROT_FASTA_OUT
        Output protein FASTA file of reported proteins
    --nucleotide_output NUC_FASTA_OUT
        Output nucleotide FASTA file of reported nucleotide sequences
    --nucleotide_flank5_output NUC_FLANK5_FASTA_OUT
        Output nucleotide FASTA file of reported nucleotide sequences with 5' flanking sequences
    --nucleotide_flank5_size NUC_FLANK5_SIZE
        5' flanking sequence size for NUC_FLANK5_FASTA_OUT
        Default: 0
    --blast_bin BLAST_DIR
        Directory for BLAST. Deafult: $BLAST_BIN
    --hmmer_bin HMMER_DIR
        Directory for HMMer
    -q, --quiet
        Suppress messages to STDERR
    --pgap
        Input files PROT_FASTA, NUC_FASTA and GFF_FILE are created by the NCBI PGAP
    --gpipe_org
        NCBI internal GPipe organism names
    --parm PARM
        amr_report parameters for testing: -nosame -noblast -skip_hmm_check -bed
    --threads THREADS
        Max. number of threads
        Default: 4
    --debug
        Integrity checks
    --log LOG
        Error log file, appended, opened on application start
    
    Temporary directory used is $TMPDIR or "/tmp"
    (class7) 




```bash
# We will generate AMRfinderplus commands for each individual genomes with the following for loop and submit it as a cluster job.
for i in /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/*.fasta; 
do 
output=`echo $i | cut -d'/' -f9 | sed 's/.fasta/.txt/g'`; 
report=`echo $i | cut -d'/' -f9 | sed 's/.fasta/_mutation_report.tsv/g'`; 
echo "amrfinder --plus --output $output -n $i --mutation_all $report --organism Klebsiella_pneumoniae";
# Comment this line to print AMRFinderPlus commands
amrfinder --plus --output $output -n $i --mutation_all $report --organism Klebsiella_pneumoniae;
done
```

- Note that you would need to specify project specific organism name with the `--organism` argument. 
- `amrfinder -l` will list possible options for this argument.

```Available --organism options: Acinetobacter_baumannii, Burkholderia_cepacia, Burkholderia_pseudomallei, Campylobacter, Citrobacter_freundii, Clostridioides_difficile, Enterobacter_asburiae, Enterobacter_cloacae, Enterococcus_faecalis, Enterococcus_faecium, Escherichia, Klebsiella_oxytoca, Klebsiella_pneumoniae, Neisseria_gonorrhoeae, Neisseria_meningitidis, Pseudomonas_aeruginosa, Salmonella, Serratia_marcescens, Staphylococcus_aureus, Staphylococcus_pseudintermedius, Streptococcus_agalactiae, Streptococcus_pneumoniae, Streptococcus_pyogenes, Vibrio_cholerae, Vibrio_parahaemolyticus, Vibrio_vulnificus```


```bash
touch AMRFinderPlus_demo.sbat
```

    (class7) 



Copy and paste these lines to AMRFinderPlus_demo.sbat file using nano:

```
#!/bin/sh
# Job name
#SBATCH --job-name=AMRFinderPlus
# User info
#SBATCH --mail-user=username@umich.edu
#SBATCH --mail-type=BEGIN,END,FAIL,REQUEUE
#SBATCH --export=ALL
#SBATCH --partition=standard
#SBATCH --account=epid582w24_class
# Number of cores, amount of memory, and walltime
#SBATCH --nodes=1 --ntasks=1 --cpus-per-task=8 --mem=40g --time=12:00:00
#  Change to the directory you submitted from
cd $SLURM_SUBMIT_DIR
echo $SLURM_SUBMIT_DIR

for i in /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/*.fasta; 
do
output=`echo $i | cut -d'/' -f9 | sed 's/.fasta/.txt/g'`; 
report=`echo $i | cut -d'/' -f9 | sed 's/.fasta/_mutation_report.tsv/g'`;
# Uncomment this line to print AMRFinderPlus commands.
#echo "amrfinder --plus --output $output -n $i --mutation_all $report --organism Klebsiella_pneumoniae";
amrfinder --plus --output $output -n $i --mutation_all $report --organism Klebsiella_pneumoniae;
done

```


```bash
sbatch AMRFinderPlus_demo.sbat
```

#### 3. Run MLST detection using mlst tool

Link to MLST tool: https://github.com/tseemann/mlst


```bash
cd /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/
```


```bash
# creating a new folder to save mlst results
mkdir mlst_demo
cd mlst_demo
```


```bash
# Activated the mlst conda environment
module load Bioinformatics mlst
```

    
        Provides Bioinformatics software.
        For more information please use:
    
            $ module help Bioinformatics
    
    



```bash
# Check if mlst tool was installed and check if dependencies looks okay: 
mlst -check
```

    [11:23:35] This is mlst 2.23.0 running on linux with Perl 5.032001
    [11:23:35] Checking mlst dependencies:
    [11:23:35] Found 'blastn' => /sw/pkgs/med/mlst/2.23.0/bin/blastn
    [11:23:35] Found 'any2fasta' => /sw/pkgs/med/mlst/2.23.0/bin/any2fasta
    [11:23:42] Found blastn: 2.14.0+ (002014)
    [11:23:42] OK.



```bash
# Check if mlst tool contain mlst typing scheme for klebsiella
mlst -list
```

    [11:23:42] This is mlst 2.23.0 running on linux with Perl 5.032001
    [11:23:42] Checking mlst dependencies:
    [11:23:42] Found 'blastn' => /sw/pkgs/med/mlst/2.23.0/bin/blastn
    [11:23:42] Found 'any2fasta' => /sw/pkgs/med/mlst/2.23.0/bin/any2fasta
    [11:23:43] Found blastn: 2.14.0+ (002014)
    pmultocida plarvae sbsec ureaplasma brachyspira psalmonis chlamydiales spneumoniae cbotulinum campylobacter_nonjejuni_8 sepidermidis campylobacter_nonjejuni_5 pfluorescens mplutonius brachyspira_5 campylobacter_nonjejuni_7 shaemolyticus csepticum neisseria bhenselae bwashoensis campylobacter_nonjejuni cronobacter staphlugdunensis spyogenes scanis helicobacter leptospira listeria_2 bsubtilis pputida cdifficile bordetella_3 mhominis_3 hparasuis ecloacae mgallisepticum_2 campylobacter_nonjejuni_9 mcatarrhalis_achtman_6 manserisalpingitidis lsalivarius bcereus wolbachia senterica_achtman_2 mycobacteria_2 brucella miowae vcholerae brachyspira_2 gallibacterium tpallidum xfastidiosa llactis_phage spseudintermedius borrelia mcanis efaecium cfreundii geotrichum taylorella oralstrep orhinotracheale ranatipestifer vibrio kaerogenes campylobacter_nonjejuni_6 klebsiella leptospira_2 ssuis sagalactiae vtapetis ypseudotuberculosis_achtman_3 szooepidemicus dnodosus yruckeri saureus blicheniformis_14 bcc mpneumoniae pmultocida_2 aeromonas mflocculare mhyorhinis pgingivalis mhyopneumoniae brachyspira_4 pacnes_3 streptothermophilus tenacibaculum cmaltaromaticum pdamselae campylobacter_nonjejuni_4 vcholerae_2 hsuis bbacilliformis bpseudomallei brachyspira_3 shominis otsutsugamushi ppentosaceus abaumannii_2 vparahaemolyticus koxytoca smaltophilia fpsychrophilum hinfluenzae sgallolyticus rhodococcus ecoli_achtman_4 bfragilis abaumannii kingella edwardsiella aactinomycetemcomitans achromobacter cperfringens aphagocytophilum diphtheria_3 mgallisepticum sinorhizobium shewanella streptomyces sdysgalactiae campylobacter_nonjejuni_3 suberis schromogenes liberibacter campylobacter_nonjejuni_2 arcobacter campylobacter paeruginosa sthermophilus efaecalis mhaemolytica hcinaedi msciuri magalactiae ecoli msynoviae mabscessus vvulnificus mcaseolyticus leptospira_3 mbovis_2



```bash
# Run mlst tool on all the CRKP assemblies in crkp_assembly folder and save the output in csv format to a file mlst.csv.
mlst --quiet --scheme klebsiella --csv /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/*.fasta > crkp_mlst.csv
```


```bash
head crkp_mlst.csv
```

    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204326.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204327.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204328.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204329.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204330.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204332.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204334.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204335.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204336.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    /scratch/epid582w24_class_root/epid582w24_class/shared_data/data/assignment_2/crkp_assembly/SRR6204337.fasta,klebsiella,258,gapA(3),infB(3),mdh(1),pgi(1),phoE(1),rpoB(1),tonB(79)
    (mlst) 

