+++
title = 'Download_fastqs'
date = 2024-05-19T01:28:02-04:00
draft = true
comments = true
weight = 1
+++


I want to download fastq files from SARS-CoV-2 Analyses and look at the variants

First, we need to find some raw data that we can work with.  After googling various combinations of SARS-CoV-2, sequencing, FASTQs I came across this paper which analyzed over 229,000 sequences:  
Farkas C, Mella A, Turgeon M, Haigh JJ. A Novel SARS-CoV-2 Viral Sequence Bioinformatic Pipeline Has Found Genetic Evidence That the Viral 3' Untranslated Region (UTR) Is Evolving and Generating Increased Viral Diversity. Front Microbiol. 2021 Jun 21;12:665041

In this paper, the authors demonstrate they have developed a bioinformatic pipeline that processes SARS-CoV-2 genome sequences in FASTA/FASTQ format and outputs a Variant Calling Format file for variant annotation and population genetic testing. Using their pipeline, the authors analyzed >229,000 viral sequences, identifying over 39,000 variants, including variants in the ORF3a gene, the 3′ untranslated (UTR) regions and recurrent ORF8 gene inactivation.  

Some interesting notes about SARS-CoV-2: 
- it's a positive single-stranded RNA virus  
- it binds through its Spike glycoprotein to angiotensin-converting enzyme 2 (ACE2) to get into the host cells
- The SARS-CoV-2 genome contains 29,811 nucleotides and codes for 29 different viral proteins
- the viral proteins consist of non-strucural proteins and accessory proteins and structural proteins like the spike, membrane and envelope proteins
- the 3' UTR is at the end of the genome that codes for the structural proteins

For a nice diagram of the genome see Figure 2 in this paper:  
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8552795/


Now that we have a source of data we can find the fastqs to download  

Using the Supplementary Table 1 from the paper, which lists all their sequencing data, I choose to look at the samples from QUEST DIAGNOSTICS  
We can search for the corresponding SRA study ID SRP267191 or BioProject PRJNA631061 in the SRA Run Selector  

I then choose the samples I wanted by checking the box by each row number then clicked Accession List

![jpg](SRA image)


Because I'm working on my local machine, I choose the 25 smallest files (I would NOT recommend this approach of choosing only the smallest or largest files for any sort of real analysis, this is just for me to be able to go through the process on my machine at home)

Ok, so I selected my files, clicked Accession List and saved that list of SRR Run IDs as: PRJNA631061_quest_diagnostics_SRR.txt

Next up, we get the fastq files using fastq dump and the list of my SRR Run IDs

First we need to prepare a project directory:
in the terminal: 
cd to your directory where you want to store this project

```sh
mkdir SARS-CoV-2
cd SARS-CoV2
mkdir code
mkdir raw-data
mkdir processed-data
mkdir results
cd code
```

To run the code below all together:
copy the code -> save it as a .sh file in your code folder -> run: bash ./01-Get_QD_FASTQs.sh
Put your SRR.txt file in raw-data

```sh
#!/bin/bash  # shebang or hashbang -- this tells the computer to use the Bash shell to run this script

# -ue stop at any error
# -x print commands upon execution
set -uex

# SRR IDs for covid samples
cat ../raw-data/PRJNA631061_quest_diagnostics_SRR.txt

# make directories to store fastqs
mkdir ../raw-data/fastqs  # first make fastq file

# we can use GNU parallel to run mkdir concurrently where the {} is replaced by each line from the PRJNA631061_quest_diagnostics_SRR.txt file (each line = SRR ID)
## we'll end up with a folder structure in raw-data like the following with one per SRR ID: raw-data/fastqs/SRR12975623
cat ../raw-data/PRJNA631061_quest_diagnostics_SRR.txt | parallel 'mkdir ../raw-data/fastqs/{}'

# Download the files from SRA
## Format of fastq-dump: fastq-dump --split-files SRR519926
## store the fastqs in each of the folders we just created in the previous step
cat ../raw-data/PRJNA631061_quest_diagnostics_SRR.txt | parallel 'fastq-dump --split-files --outdir ../raw-data/fastqs/{} {}'

# Trim adaptor reads
## We use parallel again to run fastp on each line from the input file
cat ../raw-data/PRJNA631061_quest_diagnostics_SRR.txt | parallel 'fastp --cut_tail -i ../raw-data/fastqs/{}/{}_1.fastq -I ../raw-data/fastqs/{}/{}_2.fastq -o ../raw-data/fastqs/{}/{}_1.trim.fq -O ../raw-data/fastqs/{}/{}_2.trim.fq'
# -i specifies the input file for the first read
# -I specifies the input file for the second read
# -o specifies the output file for the trimmed first read
# -O specifies the output file for the trimmed second read


# run FastQC - FastQC is used as a quality control step.  It provides overview of the quality of the reads, so we will examine the output before proceeding with the alignment
## Format of fastqc: fastqc SRR519926_1.fastq SRR519926_2.fastq
## Run fastqc on all our trimmed fastqs and store the output in raw-data/fastqs/SRR_ID
cat ../raw-data/PRJNA631061_quest_diagnostics_SRR.txt | parallel 'fastqc -o ../raw-data/fastqs/{} ../raw-data/fastqs/{}/{}_1.trim.fq ../raw-data/fastqs/{}/{}_2.trim.fq'

# Run multiqc
## MultiQC will aggregate all our results from FastQC so we can look at the quality of all our samples together
multiqc ../raw-data/fastqs -o ../results
```