Class 2 – Sequence data formats
===============================

Goal
----

- In this module, we will explore different sequence file formats 
- we will learn how to perform pattern matching using grep to find files and texts in files
- we will apply for loops to perform same repetitive tasks on different files
- We will wrap this session up by saving our code in a shell script so that we can turn our commands into a small executable software.

Using GREP for pattern matching
-------------------------------

Up until now you’ve probably accessed sequence data from NCBI by going to the website, laboriously clicking around and finally finding and downloading the data you want. 

There are a lot of reasons that is not ideal:

- It’s frustrating and slow to deal with the web interface
- It can be hard to keep track of where the data came from and exactly which version of a sequence you downloaded
- Its not conducive to downloading lots of sequence data

To download sequence data in Unix you can use a variety of unix commands such as sftp, wget, curl. In class 6, we will use third party tools that are specifically developed to download data from sequence databases.

Here, we will use curl command to download some genome assemblies from NCBI ftp location:

- Go to your class home directory (use your wd shortcut!)

- Execute the following commands to copy files for class2 to your home directory: 

```
cp -r /scratch/epid582w22_class_root/epid582w22_class/shared_data/data/class3 ./

cd class3/

ls

```

- Now get three genome sequences with the following commands:

```
curl ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/241/685/GCF_000241685.1_ASM24168v2/GCF_000241685.1_ASM24168v2_genomic.fna.gz >Acinetobacter_baumannii.fna.gz

curl ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/409/005/GCF_000409005.1_gkp33v01/GCF_000409005.1_gkp33v01_genomic.fna.gz > Kleb_pneu.fna.gz

curl ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/165/655/GCF_000165655.1_ASM16565v1/GCF_000165655.1_ASM16565v1_genomic.fna.gz > E_coli.fna.gz

```

- Decompress the gzip compressed fasta file using gzip command

```
gzip -d Acinetobacter_baumannii.fna.gz
gzip -d Kleb_pneu.fna.gz
gzip -d E_coli.fna.gz
```

These files are genome assemblies in fasta format. Fasta files are a common sequence data format that is composed of alternating sequence headers (sequence names and comments) and their corresponding sequences. Of great importance, the sequence header lines must start with “>”. These genome assemblies have one header line for each contig in the assembly, and our goal will be to count the number of contigs/sequences. To do this we will string together two Unix commands: “grep” and “wc”. “grep” (stands for global regular expression print), is an extremely powerful pattern matching command, which we will use to identify all the lines that start with a “>”. “wc” (stand for word count) is a command for counting words, characters and lines in a file. To count the number of contigs in one of your fasta files enter:


```
grep ">" E_coli.fna | wc -l
```

Try this command on other assemblies to see how many contigs they contain. 

Using for loops to perform same actions on different files
----------------------------------------------------------

OK, so now that we have a useful command, wouldn’t it be great to turn it into a program that you can easily apply to a large number of genome assemblies? Of course it would! So, now we are going to take out cool contig counting command, and put it in a shell script that applies it to all files in the desired directory.

<!--- Copy “/scratch/micro612w21_class_root/micro612w21_class/shared/fasta_counter.sh” to your current directory (Hint – use the “cp” command)-->

There will be times when you have multiple sets of files in a folder in which case it becomes cumbersome to run individual commands on each file. To simplify this task, most programming language have a concept of loops that can be employed to repeat a task/command on a bunch of files repeatedly. Here we have three fasta files for which we want to know the number of contigs in each file. We can either run the above mentioned grep command seperately on each file or use it in a "for" loop that iterates through a set of values/files until that list is exhausted. 

Try the below example of for loop, that loops over a bunch of numbers and prints out each value until the list is exhausted.

```
for i in 1 2 3 4 5; do echo "Looping ... number $i"; done
```

A simple for loop statement consists of three sections: 

1. for statement that loops through values and files
2. a do statement that can be any type of command that you want to run on a file or a tool that uses the current loop value  as an input
3. done statement that indicates completion of a do statement.

Note that the list values - (1 2 3 4 5) in the above for loop can be anything at all. It can be a bunch of files in a folder with a specific extension (\*.gz, \*.fasta, \*.fna) or a list of values generated through a seperate command that we will see later.

We will incorporate a similar type of for loop in fasta_counter.sh script that will loop over all the \*.fna files in the folder. We will provide the name of the folder through a command line argument and count the number of contigs in each file. A command line argument is a sort of input that can be provided to a script which can then be used in different ways inside the script. fasta_counter.sh requires to know which directory to look for for \*.fna files. For this purpose, we will use positional parameters that are a series of special variables ($0 through $9) that contain the contents of the command line. 

Lets take an example to understand what those parameters stands for:


```
./some_program.sh Argument1 Argument2 Argument3
```

In the above command, we provide three command line Arguments that acts like an input to some_program.sh 
These command line argument inputs can then be used to inside the scripts in the form of $0, $1, $2 and so on in different ways to run a command or a tool.

Try running the above command and see how it prints out each positional parameters. $0 will be always be the name of the script. $1 would contain "Argument1" , $2 would contain "Argument2" and so on...

Lets try to incorporate a for loop inside the fasta_counter.sh script that uses the first command line argument - i.e directory name and search for \*.fna files in that directory and runs contig counting command on each of them.

- Open “fasta_counter.sh” in nano or your favourite text editor and follow instructions for making edits so it will do what we want it to do

- Run this script in day1am directory and verify that you get the correct results. Basic usage of the script will be:

./fasta_counter.sh <directory containing files>

```
./fasta_counter.sh .
```

The "." sign tells the script to use current directory as its first command line argument($1) 

Power of Unix commands
----------------------

In software carpentry, you learned working with shell and automating simple tasks using basic unix commands. Lets see how some of these commands can be employed in genomics analysis while exploring various file formats that we use in day to day analysis. For this session, we will try to explore two different types of bioinformatics file formats: 

gff: used for describing genes and other features of DNA, RNA and protein sequences

fastq: used for storing biological sequence / sequencing reads (usually nucleotide sequence) and its corresponding quality scores



Exploring GFF files
-------------------

The GFF (General Feature Format) format is a tab-seperated file and consists of one line per feature, each containing 9 columns of data.

column 1: seqname - name of the genome or contig or scaffold

column 2: source - name of the program that generated this feature, or the data source (database or project name)

column 3: feature - feature type name, e.g. Gene, exon, CDS, rRNA, tRNA, CRISPR, etc.

column 4: start - Start position of the feature, with sequence numbering starting at 1.

column 5: end - End position of the feature, with sequence numbering starting at 1.

column 6: score - A floating point value.

column 7: strand - defined as + (forward) or - (reverse).

column 8: frame - One of '0', '1' or '2'. '0' indicates that the first base of the feature is the first base of a codon, '1' that the second base is the first base of a codon, and so on..

column 9: attribute - A semicolon-separated list of tag-value pairs, providing additional information about each feature such as gene name, product name etc.

- Use less to explore first few lines of a gff file sample.gff

```

less sample.gff

```
Note: lines starting with pound sign "#" represent comments and are used to document extra information about the features.

You will notice that the GFF format follows version 3 specifications("##gff-version 3"), followed by genome name("#Genome: 1087440.3|Klebsiella pneumoniae subsp. pneumoniae KPNIH1"), date("#Date:02/09/2017") when it was generated, contig name("##sequence-region") and finally tab-seperated lines describing features.

You can press space bar on keyboard to read more lines and "q" key to exit less command.

- Question: Suppose, you want to find out the number of annotated features in a gff file. how will you achieve this using grep and wc?

<details>
  <summary>Solution</summary>
  
```
grep -v '^#' sample.gff | wc -l
```
</details>

- Question: How about counting the number of rRNA features in a gff(third column) file using grep, cut and wc? You can check the usage for cut by typing "cut --help"

<details>
  <summary>Solution</summary>
  
```

cut -f 3 sample.gff | grep 'rRNA' | wc -l

#Or number of CDS or tRNA features?

cut -f 3 sample.gff | grep 'CDS' | wc -l
cut -f 3 sample.gff | grep 'tRNA' | wc -l

#Note: In the above command, we are trying to extract feature information from third column.

```
</details>

- Question: Try counting the number of features on a "+" or "-" strand (column 7).

Now, let's use what we learned about for loops and creating shell scripts above to create a script called "feature_counter.sh" This script will take as input a directory and will search for all gff files in the directory. It will calculate and output the number of tRNA features in the gff.   

- Open “feature_counter.sh” in nano or your favourite text editor and follow instructions for making edits so it will do what we want it to do

- Run this script in day1am directory and verify that you get the correct results. Basic usage of the script will be:

./feature_counter.sh <directory containing gff files>

```
./feature_counter.sh .
```


Some more useful one-line unix commands for GFF files: [here](https://github.com/stephenturner/oneliners#gff3-annotations)