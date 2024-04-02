+++
title = 'Downloading Data from SRA'
date = 2024-03-07T19:15:13-05:00
draft = false
comments = true
+++


So you found an interesting publication or project and you want to download the data
Let's download some data from GEO

I studied GBMs in grad school, so I came across this interesting publication:
**Longitudinal genomic study of response to anti-PD-1 immunotherapy in GBM**, Zhao et al, Nature Med 2019

The paper says: All the sequencing data have been deposited in SRA PRJNA482620. Processed data and basic association analyses will be made available upon request.

Given that information, my first step is to go to SRA and seach for this project code  
And we get something that looks like this:

![SRA 1](../images/SRA_screenshot_1.png)

But that can be a little overwhelming to look at with each sample shown separately.  So I will click on: 
"Send results to SRA run selector" (highlighed in yellow in image above)

And now we can see the samples in table format which can give us more information in one place  
We can also download the metadata from here

![SRA 2](../images/SRA_screenshot_meta_data.png)

You could download the files using the sratoolkit and fastq-dump or fasterq-dump, but here I will use sra-explorer to create a bash script to download the files

Now I'm going to go to sra-explorer and paste in the project ID: PRJNA482620
Then in the Filter results box, I'm going to put in "RNA" to get the RNA-seq files
Then click the blue "Add 34 to collection" box
Then go to my cart at the top right of the page

Then I'm going to click on:
Bash script for downloading FastQ files and download that file
You should get a file named: sra_explorer_fastq_download.sh

Ok so now I'm ready to download the data to our system

So I will move to the directory where I want to store these files:
cd Bioinformatics/PRJNA482620/fastqs
../code/sra_explorer_fastq_download.sh

![SRA 3](../images/curl_data.png)

The message here: ```#!/usr/bin/env: No such file or directory``` is most likely happening because I don't have env command installed on my system  
But even so I can see the files are actively being downloaded.  So now we wait...
![Coffee](../images/coffee_break.png)


Ok, so it looks like everything has been downloaded, but we can do a quick check to make sure we have all the expected files:

Get list of run IDs from the SRA run table I downloaded

```
# In the SraRunTable file get the RNAseq files (grep for RNA-seq) then get the Run ID by only taking the first column of the SraRunTable file
cat raw-data/SraRunTable.txt | grep RNA-Seq | cut -d ',' -f 1 > SRR_files.txt

# Append _RNA-seq_of_Homo_Sapiens_1.fastq.gz and _RNA-seq_of_Homo_Sapiens_2.fastq.gz to the Run ID
awk '{print $0 "_RNA-seq_of_Homo_Sapiens_1.fastq.gz"}' SRR_files.txt > SRR_all_files.txt
awk '{print $0 "_RNA-seq_of_Homo_Sapiens_2.fastq.gz"}' SRR_files.txt >> SRR_all_files.txt
```

Iterate over each filename in the text file I just created and check if that file exists in the fastq directory and write any missing files to the text file "missing_files.txt"
```
directory="fastqs"
while IFS= read -r filename; do

    # Check if the filename exists in the directory
    if [ -e "$directory/$filename" ]; then
        echo "$filename exists in $directory"
    else
        echo "$filename does not exist in $directory"
        echo "$filename" >> missing_files.txt
    fi
done < SRR_all_files.txt
```



