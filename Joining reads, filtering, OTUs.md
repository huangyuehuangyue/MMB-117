Now we have the quality trimmed sequence reads. Remember, garbage in, garbage out. 

# Joining forward and reverse reads

Next we join the paired end reads with program Pear. First you'll need to install it. 

```
module load bioconda/3

conda create -n pear pear
```
Now you have created `pear`environment. Activate it with
```
source activate pear
```
```
pear -f R1read.fastq_trim -r R2read.fastq_trim -o sample_pear_assembled.fastq
```

After this lets check the length distribution of the reads with program Prinseq

```
prinseq-lite.pl -fastq *fastq  -stats_all
```
And this is so much easier with scripts. Copy script `pear_batch.py` from Jenni's shared folder. How did you change rights to execute the file?

run script with
```
pear_batch.py mapping.txt
```
How much of the reads assembled? How about the negative control?

## Filtering reads and transforming them to fasta format
From now on we will use usearch tools from Robert Edgar. Read more at 

```
usearch --fastq_filter sample1_pear.assembled.fastq --threads 2 --fastq_maxee 1 --relabel @ --fastq_minlen 350 --fastaout sample1_pear.assembled.usearch.trimmed.fasta -threads 2
```
And with script: 
```
#!/bin/bash

while read i
do
        arr=($i)
        usearch --fastq_filter ${arr[0]}_pear.assembled.fastq --threads 2 --fastq_maxee 1 --relabel @ --fastq_minlen 350 --fastaout ${arr[0]}_pear.assembled.usearch.trimmed.fasta -threads 2
done < mapping.txt
```

## Add sample name to sequences
In order to analyze OTUs in samples we need to combine all of the sequence data. before this let's add sample identiier to each sequence file (pear assembled, trimmed sequences) with command `sed`

```
sed "s/>@*/>barcodelabel=sample1;read=/g"  sample1_pear.assembled.vsearch.trimmed.fasta >  sample1_renamed
```
Finally we can join our sequences together

´´´
cat `*_renamed` > all.assembled.trimmed.renamed.fasta
´´´
## Find unique read sequences and abundances
```
usearch -fastx_uniques all.assembled.trimmed.renamed.fasta -sizeout -relabel Uniq -fastaout uniques.fasta
```

In case this is too memory consuming we can run its as batch job. 

Nano fastx.batch

```
#!/bin/bash -l
#SBATCH -J usearch
#SBATCH -o usearch_out_%j.txt
#SBATCH -e usearch_err_%j.txt
#SBATCH -t 01:00:00
#SBATCH -n 1
#SBATCH -p serial
#SBATCH --mem=100
#

usearch -fastx_uniques all.assembled.trimmed.renamed.fasta -sizeout -relabel Uniq -fastaout uniques.fasta
```
After it is done, you can submit it to the SLURM system with `sbatch` command

```
sbatch fastx.batch
```
You can check the status of your job with:

```
squeue -l -u $USER
```
View the output file `uniques.fasta` with less. What does the `size=` refer to? What is the largest size? Exit less with `q`

## Make OTUs

```
usearch -cluster_otus uniques.fasta -otus otus.fasta -relabel OTU
```
How many OTUs did you have? How about chimeras?
