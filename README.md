# YAAP
Yet Another Amplicon denoising Pipeline (YAAP), is a pipeline to analyse 
metabarcoding amplicon data. It performs QC, adapter removal, denoising and 
ZOTU table construction.

This is a fork to tidy up the documentation and bash formatting so I could understand it better.
Original repo at https://github.com/CristescuLab/YAAP .

Table of Contents
=================

   * [YAAP](#yaap)
      * [Dependencies](#dependencies)
      * [Installation](#installation)
      * [Usage](#usage)
      * [ISSUES](#issues)

## Dependencies
This pipeline makes use of the following programs:
1. cutadapt ([https://cutadapt.readthedocs.io/en/stable/index.html](
https://cutadapt.readthedocs.io/en/stable/index.html))
2. vsearch ([https://github.com/torognes/vsearch](
https://github.com/torognes/vsearch))
3. seqkit ([https://bioinf.shenwei.me/seqkit/](
https://bioinf.shenwei.me/seqkit/))
4. pear ([https://cme.h-its.org/exelixis/web/software/pear/](
https://cme.h-its.org/exelixis/web/software/pear/))
5. usearch ([https://www.drive5.com/usearch/](
https://www.drive5.com/usearch/))
6. fastqc ([https://www.bioinformatics.babraham.ac.uk/projects/fastqc/](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))

This git contains the binaries (executables) of pear, seqkit, and usearch, 
however, vsearch and cutadapt have to be installed locally.

## Installation

1. All the dependencies for this pipeline (except usearch) can be easily installed using conda.

```
# grab the conda environment specification file
wget 'https://raw.githubusercontent.com/ilevantis/YAAP/master/yaap_env.yml'
# install the specified dependencies with conda
conda env create -f yaap_env
```

2. Now install the appropriate version of usearch from https://www.drive5.com/usearch/ . 

3. Test that all dependencies work:
```
cutadapt -h
vsearch -h
pear -h
seqkit -h
usearch 
```

4. Put a copy of ASV_pipeline.sh into your bin folder and make it executable:
```
cd ~/bin
wget 'https://raw.githubusercontent.com/ilevantis/YAAP/master/ASV_pipeline.sh'
chmod +x ASV_pipeline.sh
```

## Usage
YAAP has a single bash script called ASV_pipeline.sh. It requires 8 arguments:
1. File with a list of fileset prefix (one per line)
2. Forward primer sequence
3. Reverse primer sequence
4. Primer name
5. Prefix of the output
6. Number of cpus to use
7. Minimum amplicon length
8. Maximum amplicon length
9. Unoise minimum abundance parameter

This pipeline assumes that your files are names fileset_prefix_R1.fastq.gz and 
fileset_prefix_R2.fastq.gz for all the samples. It also assumes that you have 
demultiplexed your samples.

As an example, let's assume that we have two samples named MC1 and MC2. Your 
demultiplexed fastq files are `MI.M03555_0320.001.N701N517.MC1_R1.fastq.gz`, 
`MI.M03555_0320.001.N701N517.MC1_R2.fastq.gz`, 
`MI.M03555_0320.001.N702N517.MC2_R1.fastq.gz`,
`MI.M03555_0320.001.N702N517.MC2_R1.fastq.gz`.
 You will need to create a file that looks like this:
 ```
MI.M03555_0320.001.N701N517.MC1
MI.M03555_0320.001.N702N517.MC2 
 ```

The args 2 and 3 are the forward and reverse primer sequences. Forward primer 
is written as expected (5'-3'). Reverse primer is also written in the 5'-3'
orientation - i.e. what you would see at the beginning of a read derived 
from the reverse strand of the amplicon.

For the following dsDNA amplicon:
```
dsDNA:
5'-ACTAGgctgactgTTTAG-3'
3'-TGATCcgactgacAAATC-5'

read from forward strand:
ACTAGgctgactgTTTAG

read from reverse strand:
CTAAAcagtcagcCTAGT
```

Forward primer = ACTAG
Reverse primer = CTAAA

**Example command:**

The sample fastq files are listed in `file_list.txt`. The primers used were 
 `GGWACWGGWTGAACWGTWTAYCCYC` as forward, and `TAAACTTCAGGGTGACCAAAAAATCA` as 
 reverse. Running on a machine with 28 cpus. You can run the pipeline as:
```
bash ~/YAAP/ASV_pipeline.sh file_list.txt \
GGWACWGGWTGAACWGTWTAYCCYC TAAACTTCAGGGTGACCAAAAAATCA \
COI test 28 293 333 4
```

This call of the pipeline will focus on reads that contained the primer 
sequences, that were longer than 126 (we focused on reads that are longer than 
(min_len/2) - 20) in either direction (forwards and reverse), and which 
assembly shoud be between 293bp and 333 (around expected length of the Leray 
amplicon). The denoising would filter out reads with lower abundances than 4.
It will create an output folder with the name of the primer (COI in 
the example) and append test to the output files. In the example, the output 
folder will be called `test_outputCOI`.


## ISSUES
If you find any bugs related with the pipeline, please open an issue in the the
github repo, and add some traceback (error) information.
