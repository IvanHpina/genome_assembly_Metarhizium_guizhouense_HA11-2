# **Pipeline Assembly and annotation of the _Metarhizium guizhouense_ HA11-2 fungus**

## Introduction

This pipeline describes the programs, steps, results, and considerations that will be
used during _de Novo_ assembly and annotation of the fungal genome
_Metarhizium guizhouense_ HA11-2.

The pipeline is divided into several sections, taking into account the main
processes that are carried out

1. Analysis of the quality of the sequences and filtering (trimming) of the sequences
of low quality.
2. _de Novo_ assembly of the genome with the reads already filtered and filtering of the
polluting and low coverage contigs
3. Scaffolding and gap filling in the assembly.
4. Structural and functional annotation.

## 1. Analysis of the quality of the sequences and filtering (trimming) of low quality sequences.
### 1.1 Quality of sequences

Sequence analysis is an important part of the process, as it always
There are errors in mass sequencing. For this particular case,
performed genome sequencing using **Illumina HI-Seq 2500** using
paired sequencing or pairend (sequencing of the two chains) using
the V4 chemistry kit for general library, with a depth of 150 x,
126 bp.
Two fastq type files called R1 and R2 (forward and reverse) are sent. For
to analyze the quality, the **Fastqc** program is used on the command line, it is
You can access the manual and download the program through this [link](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
or you can install by `conda install fastqc` and to see the options
`fastqc -h`.

On the command line, type the following for this process
```bash
fastqc /home/chemistry/Desktop/Labgenmolh_bioinfo/Metarhizum_guizhouense/00_rawdata/*fastq.gz
```
The result is displayed in **.html**.
Within the **.html** file there are several quality graphs, one of the most important
It is the first graph, this shows in a box and whisker graph the quality
of phred 33 of each nucleotide of all reads per position, for example shown
the graph of ![R1](/images/2023/05/R1.png) (forward) and ![R2](/images/2023/05/R2.png) (reverse).

It also shows a table with the total number of reads, %GC, length among other data that generates several files, of which the most important are the
The quality of these data is considered good, most of the quality of the reads
It is in the green zone.

## 1.2 Filtering (trimming) of the sequences

After the quality analysis, a trimming was carried out, that is, those
reads on average do not meet the quality criteria, this is to improve the
quality of the reads and improve the assemblies, in addition the sequences can be removed
of the adapters used in sequencing, in this case the raw data
They already came without adapters.
For this, the **trimmomatic** program is used, in this [link](http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf "trimmomatic manual") you will find the manual.
  In the command line we type the following:
```bash
trimmomatic PE -phred33 00_rawdata/HAII_2_DNA_Sec_R1.fastq.gz 00_rawdata/HAII_2_DNA_Sec_R2.fastq.gz 02_trimming/HAII_2_DNA_Sec_R1_PE.trimming.fastq.gz 02_trimming/HAII_2_DNA_Sec_R1_unPE.trimming.fastq.gz 02_trimming/HAII_2_DNA_Sec_R2_PE.trimming.fastq.gz 02_trimming/HAII_2_DNA_Sec_R2_unPE.trimming.fastq.gz
SLIDING WINDOW:4:25 MINLEN:40
```
4 files will be obtained, the paired ones and the non-paired ones, the ones that interest us will be
those paired, these files underwent a quality analysis with `fasqc`.
Quality charts are compared.

## 2. _de Novo_ assembly of the genome with the reads already filtered and filtering of the
contigs
### 2.1 _de Novo_ assembly of the genome

With the paired readings (filtered by quality or not filtered) we proceed to carry out
the _de Novo_ assembly, for this the **SPAdes** program will be used. This program what it does
is to use a certain length of the read, this is K-mer or k-mer, this is designated at the time of running
the program, you can put several k-mer, in this case three were used: 21, 33, 55.

On the command line, type the following:
```bash
spades.py -1 02_trimming/HAII_2_DNA_Sec_R1_PE.trimming.fastq.gz -2 02_trimming/HAII_2_DNA_Sec_R2_PE.trimming.fastq.gz -k 21,25,33,47,55,67,71,83,99,111 --careful -o 03_assembly_ allkmer_filter -t 6
```
This process requires a lot of computational capacity, so it is recommended not to
run other processes concurrent to this one.

Finishing the process several files are generated, of which the most important are
the contigs.fasta and the scaffolds.fasta. which can be analyzed to determine
the number of contigs `grep ">" contigs.fasta or scafolds.fasta | wc -l`
later a graph of the contigs or scaffolds vs their coverage will be made to remove
all those polluting contigs.

## 2.2 Filtering contaminants and low coverage contigs

The problem with massive sequencing is the number of samples that are run, this can
cause cross contamination, in addition, from the beginning it can carry genetic material of bacteria
or viruses and the algorithms of the **SPAdes** program generate small contigs with coverage
less than the smallest k-mer used, in this case 21.

Therefore, those contigs with low coverage and size less than 500 bp will be removed,
First, a graph of contigs vs. coverage must be made, which can be done in
excel, in this [link][de5b4ec4] come the instructions to make the graph
from the .fasta files.

   [de5b4ec4]: https://tinyurl.com/CVLexcel "CVL plot"

The filtering of the contigs must be done manually, those contigs that do not
they comply with the aforementioned characteristics and later a new graph is made.
They are compared with each other and differences are observed.
