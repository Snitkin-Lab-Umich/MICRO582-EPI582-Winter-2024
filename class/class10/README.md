Class 10 – Intro to RStudio
=============================================

Goals
----
- Review RStudio interface
- Review important variable types
- Learn how to read in genomic analysis files and make plots


The RStudio Interface and setting up an RProject
------------------------------------------------
Check out this Data Carpentry overview of the [RStudio interface](https://datacarpentry.org/R-genomics/00-before-we-start.html)

In order for us to all work in the same environment we are going to create an RProject. RProjects are a nice way to organize all of your scripts and data for a project in a self-contained work area. To create an RProject for the class:

1. Under the File menu, click on New project, choose New directory, then Empty project
2. Enter a name for this new folder, and choose a convenient location for it. This will be your working directory for the rest of the course (e.g., ~/epid5)
3. Confirm that the folder named in the Create project as a sub-directory of box is where you want the working directory created. Use the Browse button to navigate folders if changes are needed.
4. Click on “Create project”
5. Under the Files tab on the right of the screen, click on New Folder and create a folder named data within your newly created working directory. (e.g., ~/data-carpentry/data)
6. Create a new R script (File > New File > R script) and save it in your working directory (e.g. class_11_introR.R)


Creating variables, performing operations and applying functions
----------------------------------------------------------------
Check out this Data Carpentry overview of the [basic R syntax](https://datacarpentry.org/R-genomics/01-intro-to-R.html)

Let's start out by briefly reviewing the basics of working in R. 

```
#Let's create some variables using the <- operator
a <- 2
b <- 7

#Now let's perform an operation on our two variables
c <- a+b

#Finally, let's apply a function to a variable
sqrt_c <- sqrt(c)

#If you ever want to learn how to use a function, use the ?
?sqrt
```

Next, let's look at working with vectors.

```
#Let's create a vector of genome lengths
glengths <- c(4.6, 3000, 50000)

#Next let's ask R what type of variable this is
class(glengths)

#Now, let's see how to get elements from this vector by indexing
glengths[1] #1st element
glengths[2] #2nd element
glengths[3] #3rd element

#OK - let's create another vector with the names of genomes
species <- c("ecoli", "human", "corn")
class(species)

#Finally, let's make the association between these two vectors explicit by naming our genome lengths
names(glengths) <- species
species

#Now that we've named our vector, we can index by position or name
glengths['human']
```

Finally, let's learn about how to get subsets of data by indexing with vectors or logicals.

```
#Let's start by using a special variable in R containing letters
LETTERS
ten_letters <- LETTERS[1:10];

#We saw before how to pull out one element, now let's pull out
ten_letters[c(1,2,3)]
ten_letters[c(1,9)]

#A quicker way to pull sequential elements is by using a colon
ten_letters[1:3]

#Next, let's create a vector of numbers to play with
nums <- 1:10

#Now we are going to use logical operators to subset, but first, let's learn about logicals
nums == 1
nums > 5
nums < 5
nums != 10
nums >= 5
nums <= 5

#We can use logicals to index vectors, as long as the logical is the same length
ten_letters[nums == 1]

ten_letters[nums < 5]

ten_letters[nums >= 5]
```

Reading in and parsing a gff file
---------------------------------
In previous sessions we used Unix commands to explore gff files. Now let's work with a gff file in R!

```
# Read in gff file
gff = read.table('class10/SRR5244781_contigs.gff',
                 sep = "\t", #tab delimited file
                  comment.char = "#", #define comment character and ignore those lines
                  quote = "", #tells R no quotes, so the file is parsed correctly
                  header = F)#tells R no header
 
# Examine the structure of the gff variable
str(gff)

# Rename columns
colnames(gff) = c('seqname','source','feature','start','end','score','strand','frame','attribute')

# Examine the structure of the gff variable
str(gff)

# Look at the head of the gff file
head(gff)

#Count the number of each type of feature (it's a bit easier than in unix :))
table(gff$feature)

# Get the gene lengths
gene_lengths = gff$end - gff$start

# Plot a histogram of the gene lengths
hist(gene_lengths,
     breaks = 100, # 100 cells
     xlab = 'Gene Length (bp)', # change x label
     main = '') # no title
     
# Plot a histogram of the gene lengths less than 5Kb
hist(gene_lengths[gene_lengths < 5000],
     breaks = 100, # 100 cells
     xlab = 'Gene Length (bp)', # change x label
     main = '') # no title

# Look at the feature types with length greater than 2 Kb
table(gff$feature[gene_lengths > 2000])

```

Exercise: Count how many genes are on the +/- strands? 

<details>
  <summary>Solution</summary>  
  
```
table(gff$strand)
```

</details>

Exercise: Plot a histogram of the length of genes on the + strand

<details>
  <summary>Solution</summary>  

```
hist(gene_lengths[gff$strand == "+"],
     breaks = 100, # 100 cells
     xlab = 'Gene Length (bp)', # change x label
     main = '') # no title
```

</details>


Plotting a heatmap of AMR genes from AMRfinderPlus
------------------------------------------
In class 8 we examined the results of AMRfinderPlus by writing a shell script to write a summary to the command line. Here we will instead visualize the results in R to examine the differences in AMR gene presence in environmental isolates versus human clinical isolates.

To get the data ready for plotting in R were wrote the R script 'make_amr_finder_mat.R' which goes through the results from AMRfinderPlus. Below are commands to read in our summary data and plot a heatmap.

```
#Read in data and make plots
#Load pheatmap
install.packages('pheatmap')
library(pheatmap)

#Read in gene matrix
amr_mat <- read.table('class10/amr_finder_gene_mat.txt',
                      sep = "\t",
                      quote = "",
                      check.names = F)

#Make heatmap without annotation
pheatmap(amr_mat,
         color = c('white', 'black'),
         legend = F)


#Load in annotation on isolate source and antibiotic class
source_annots = read.table('class10/kpneumo_source.tsv',row.names=1)
colnames(source_annots) = 'Source'


class_annots <- read.table('class10/amr_gene_subclass.txt', row.names = 1);
colnames(class_annots) = 'Abx subclass'


#Make heatmap with annotations
pheatmap(amr_mat,
         annotation_row = source_annots,
         annotation_col = class_annots,
         color = c('white', 'black'),
         legend = F)
  
```
