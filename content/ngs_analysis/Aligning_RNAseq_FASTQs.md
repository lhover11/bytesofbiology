+++
title = 'Aligning_RNAseq_FASTQs'
date = 2024-03-12T21:40:05-04:00
draft = true
comments = true
+++


## Aligning RNAseq data
For RNAseq reads, we need to use a splice-aware aligner designed to map the same read across several exon/exon junctions
One splice-aware aligner is hisat2


## Get our human reference genome files
### Download fasta file from UCSC
https://hgdownload.soe.ucsc.edu/downloads.html
Human >> Dec. 2013 (GRCh38/hg38) >> Genome sequence files and select annotations
Get the path for the fasta file
wget https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz

### Download the genome annotation file (GTF format)
This can be found in the genes folder
wget https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/genes/hg38.knownGene.gtf.gz

# Uncompress the files
gunzip hg38.fa.gz
gunzip hg38.knownGene.gtf.gz

# Build the genome index.
screen -S index # start a screen session so we don't lose our progress if something happens  since this will take a while
hisat2-build homo_sapiens/hg38.fa hg38