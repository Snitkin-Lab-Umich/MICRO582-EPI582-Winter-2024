Class 9 – Reference based variant calling
=============================================

Goal
----

- In this class, we will run Snippy which is a microbial variant calling pipeline on a sample to find out the difference between our sample and the reference genome.
- Look at various outputs of Snippy to explore these variants and learn what they mean.
- Visualize these variants in IGV which is a great visualization tool to put the variant calling steps in perspective.

At the start of this course, we performed some quality control steps on our sequencing data to make it clean and usable for various downstream analysis. One of the downstream sequence analysis involves finding out the differences between the sequences sample reads and the reference genome. The downstream sequence analysis involves various steps and tools that are used to transform one file into another while also extracting useful information.

A typical variant calling analysis requires:
- Indexing: Preparing your reference genome for alignments.
- Read Mapping: mapping sequenced reads to the reference genome using a read mapper
- Post-processing alignments: converting alignment formats to a binary format.
- Varuiant calling: calling variants/differences between the reference genome and our sample using variant callers
- Variant annotation: annotating these variants to learn their effects 

Here is a visual representation of these steps:

![var_call_steps](detailed_var_call.png)

