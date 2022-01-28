# This is a modified version of our COVIDSeqSEassembly workflow to handle shorter (35bp) reads:

# COVIDSeqSEassembly:

preprocessing and assembly workflows for Illumina data prepped following the COVIDSeq protocol

## COVIDSeqSEassembly.wdl
The COVIDSeqSEassembly.wdl workflow was developed for the preprocessing and assembly of Illumina COVIDSeq sequencing of SARS-CoV-2 to be run on the GCP Terra platform. It takes single-end short read fastq files as input.

The columns for the input data table in Terra should be arranged as:

1. entity:sample_id (column of sample names/ids). If there is more than one data table in the Terra Workspace, add a number after the word sample (e.g. entity:sample2_id).
2. fastq - This is the google bucket path to the fastq file.

Also needed as input (these should be saved as Workspace data and included in Terra input field for the workflow): 
1. covid_genome - the path to the google bucket directory containing the SARS-CoV-2 reference genome fasta
2. covid_annotation - the path to the google bucket directory containing the SARS-CoV-2 reference genome gff annotation file
3. V3arctic - the path to the google bucket directory containing a bed file with the primers used for amplicon sequencing

The COVIDSeqSEassembly.wdl workflow will:

1. Use Trimmomatic and bbduk to quality filter, trim, and remove adapters from raw fastq files
2. Run FastQC on both the raw and cleaned reads
3. Align reads to the reference genome using bwa and then sort the bam by coordinates using Samtools
4. Use iVar trim to trim primer regions and then sort the trimmed bam by coordinates using Samtools
5. Use iVar variants to call variants from the trimmed and sorted bam
6. Use iVar consensus to call the consensus genome sequence from the trimmed and sorted bam
7. Use Samtools flagstat, stats, and coverage to output statistics from the bam
8. Renames consensus sequences to CO-CDPHE{sample_id}
9. Calculates percent coverage using python script available on the CDPHE github page


External tools used in this workflow were from publicly available Docker images:
1. General untilities docker images: theiagen/utility:1.0 and mchether/py3-bio:v1
2. Trimmoatic: http://www.usadellab.org/cms/?page=trimmomatic
   docker image: staphb/trimmomatic:0.39
3. bbduk from the bbtools suite: https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbduk-guide/
   docker image: staphb/bbtools:38.76
4. FastQC: https://github.com/s-andrews/FastQC and https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
  docker image: staphb/fastqc:0.11.9
5. BWA: Li H. and Durbin R. (2010) Fast and accurate long-read alignment with Burrows-Wheeler Transform. Bioinformatics, Epub. [PMID: 20080505]
  docker image: broadinstitute/viral-core:latest
6. iVar: https://github.com/andersen-lab/ivar
  docker image: andersenlabapps/ivar:1.3.1
7. Samtools: Danecek P, Bonfield JK, Liddle J, Marshall J, Ohan V, Pollard MO, Whitwham A, Keane T, McCarthy SA, Davies RM, Li H, Twelve years of SAMtools and BCFtools, GigaScience (2021) 10(2) giab008 [33590861]
  docker image: staphb/samtools:1.10
