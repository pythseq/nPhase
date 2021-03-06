# nPhase
Ploidy agnostic phasing pipeline and algorithm

![Alt text](/img/nPhasePipeline.jpg?raw=true "nPhase Pipeline")

nPhase is a ploidy agnostic tool developed in python which predicts the haplotypes of a sample that was sequenced by both long and short reads by aligning them to a reference. It should work with any ploidy.

# Quick-start

[![Anaconda-Server Badge](https://anaconda.org/oakheart/nphase/badges/installer/conda.svg)](https://conda.anaconda.org/oakheart)

If you have bioconda you can install nPhase by running the following commands in your terminal:

```
conda create -n polyploidPhasing python=3.8
conda activate polyploidPhasing
conda install -c oakheart nphase
```

Then you can phase your data with the following command:

```
nphase pipeline --sampleName Individual_1 --reference /path/to/Individual_referenceGenome.fasta 
                --R1 /path/to/Individual_1_shortReads_R1.fastq.gz --R2 /path/to/Individual_1_shortReads_R2.fastq.gz
                --longReads /path/to/Individual_1_longReads.fastq.gz --longReadPlatform [ont|pacbio]
                --output /path/to/outputFolder
```


# Installation

## With bioconda

Install bioconda and set the correct channels by following the first two steps here: https://bioconda.github.io/user/install.html

Then you can create a new environment and install nPhase with the following commands:
```
conda create -n polyploidPhasing python=3.8
conda activate polyploidPhasing
conda install -c oakheart nphase
```

## Without bioconda

### Pre-requisites

You will have to install the following software before nPhase:

[Python ≥3.8](https://www.python.org/downloads/)

[bwa mem](https://github.com/lh3/bwa)

[GATK 4.0](https://github.com/broadinstitute/gatk)

[samtools v1.9](https://github.com/samtools/samtools/tree/1.9)*

[ngmlr](https://github.com/philres/ngmlr)

\*Currently, installing samtools v1.10 or higher will cause an error to occur due to [a bug in the way ngmlr handles MAPQ scores](https://github.com/philres/ngmlr/issues/83).

### Installation via PyPI

You can now install nPhase via

`pip install -U nPhase`

# Usage

There are two main ways to run nPhase:

`nphase pipeline` will run the entire pipeline from start to finish and requires the following inputs:

```
nphase pipeline --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
                --longReadPlatform {ont,pacbio} --R1 SHORT_READ_FILE_R1 --R2 SHORT_READ_FILE_R2
```
Optional parameters are described further down.

`nphase algorithm` will only run the phasing algorithm, it requires inputs generated by `nphase pipeline`. This is useful if you want to test different paramaters on your dataset after having generated the pre-processed files once. Here are the inputs required by `nphase algorithm`:

```
nphase algorithm --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
                 --contextDepth CONTEXT_DEPTHS_FILE --processedLongReads VALIDATED_SNP_ASSIGNMENTS_FILE
```

Optional parameters are described further down.

# nphase partial

Alternatively, if you already mapped your short reads to a reference, or your long reads, or already variant called your mapped short reads, you can try to use

`nphase partial` which will run only the parts of the pipeline that you need to run. So for example if you provide it with a vcf file, then it will only try to map the long reads. If you provide it with mapped short reads and mapped long reads, it will only variant call the short reads, etc. This is not recommended since I can't control what you input but it can save people time or allow for more creative uses of nPhase.

Here are the use cases of `nphase partial`:

You have mapped long reads and a vcf file of your short reads:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --vcf VCF_FILE --mappedLongReads MAPPED_LONG_READ_FILE
```
You have mapped long reads and mapped short reads, but not vcf:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --mappedLongReads MAPPED_LONG_READ_FILE --mappedShortReads MAPPED_SHORT_READ_FILE
```
You have mapped long reads, but no mapped short reads and no vcf:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --mappedLongReads MAPPED_LONG_READ_FILE --R1 SHORT_READ_FILE_R1 --R2 SHORT_READ_FILE_R2
```
You have a short read vcf, but no mapped long reads:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --vcf VCF_FILE --longReadPlatform {ont,pacbio}
```
You have mapped short reads, but no mapped long reads and no vcf:
```
nphase partial --sampleName SAMPLE_NAME --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE
               --mappedShortReads MAPPED_SHORT_READ_FILE --longReadPlatform {ont,pacbio}
```

# Parameters

```
nphase pipeline [-h] [--threads [THREADS]] [--maxID [MAXID]] [--minOvl [MINOVL]] [--minSim [MINSIM]] [--minLen [MINLEN]] --sampleName SAMPLE_NAME
                       --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE --longReadPlatform {ont,pacbio} --R1 SHORT_READ_FILE_R1 --R2
                       SHORT_READ_FILE_R2
or
nphase algorithm [-h] [--threads [THREADS]] [--maxID [MAXID]] [--minOvl [MINOVL]] [--minSim [MINSIM]] [--minLen [MINLEN]] --sampleName SAMPLE_NAME
                        --reference REFERENCE --output OUTPUT_FOLDER --longReads LONG_READ_FILE --contextDepth CONTEXT_DEPTHS_FILE --processedLongReads
                        VALIDATED_SNP_ASSIGNMENTS_FILE

positional arguments:
    pipeline            Run the entire nPhase pipeline on your sample
    algorithm           Only run the nPhase algorithm. NOTE: This will require files generated by running the pipeline mode

arguments always required:
  --sampleName STRAINNAME
                        Name of your sample, ex: "Individual_1"
  --reference REFERENCE
                        Path to fasta file of reference genome to align to, ex: /home/reference/Individual_reference.fasta
  --output OUTPUTFOLDER
                        Path to output folder, ex: /home/phased/
  --longReads LONGREADFILE
                        Path to long read FastQ file, ex: /home/longReads/Individual_1.fastq.gz

additional arguments required by nphase pipeline:
  --longReadPlatform {ont,pacbio}
                        Long read platform, must be 'ont' or 'pacbio'
  --R1 SHORTREADFILE_R1
                        Path to paired end short read FastQ file #1, ex: /home/shortReads/Individual_1_R1.fastq.gz
  --R2 SHORTREADFILE_R2
                        Path to paired end short read FastQ file #2, ex: /home/shortReads/Individual_1_R2.fastq.gz

additional arguments required by nphase algorithm:
  --contextDepth CONTEXTDEPTHSFILE
                        Path to context depths file, ex: /home/phased/Individual_1/Overlaps/Individual_1.contextDepths.tsv
  --processedLongReads VALIDATEDSNPASSIGNMENTSFILE
                        Path to validated long read SNPs, ex:
                        /home/phased/Individual_1/VariantCalls/longReads/Individual_1.hetPositions.SNPxLongReads.validated.tsv

additional arguments potentially required by nphase partial:
  --mappedShortReads MAPPEDSHORTREADS
                        Path to mapped, sorted short read file in BAM format, ex: /home/phased/Individual_1/Mapped/shortReads/Individual_1.final.bam
  --vcf VCFFILE         Path to VCF file (must be generated with GATK --ploidy=2), ex:/home/phased/Individual_1/VariantCalls/shortReads/Individual.vcf
  --mappedLongReads MAPPEDLONGREADS
                        Path to mapped long read file in SAM format and should be filtered to keep split reads (flag 260), ex: /home/phased/Individual_1/Mapped/longReads/Individual_1.sorted.sam
  --longReadPlatform {ont,pacbio}
                        Long read platform, must be 'ont' or 'pacbio'
  --R1 SHORTREADFILE_R1
                        Path to paired end short read FastQ file #1, ex: /home/shortReads/Individual_1_R1.fastq.gz
  --R2 SHORTREADFILE_R2
                        Path to paired end short read FastQ file #2, ex: /home/shortReads/Individual_1_R2.fastq.gz

optional arguments:
  -h, --help            show this help message and exit
  --threads [THREADS]   Number of threads to use on some steps, default 8
  --maxID [MAXID]       MaxID parameter, determines how different two clusters must be to prevent them from merging. Default 0.05
  --minOvl [MINOVL]     minOvl parameter, determines the minimal percentage of overlap required to allow a merge between two clusters that have fewer than
                        100 heterozygous SNPs in common. Default 0.1
  --minSim [MINSIM]     minSim parameter, determines the minimal percentage of similarity required to allow a merge between two clusters. Default 0.01
  --minLen [MINLEN]     minLen parameter, any cluster based on fewer than N reads will not be output. Default 0

```

### Paper

[nPhase: An accurate and contiguous phasing method for polyploids](https://www.biorxiv.org/content/10.1101/2020.07.24.219105v1)


### Media

Online lightning talk I gave about nPhase for an Oxford Nanopore event [5:44]: [link](https://drive.google.com/file/d/16quLufiNhICXqmAAGRYEIy1MivWVMXVI/view?usp=sharing)

### Misc

Current recommendations for default parameters are at least 20X coverage per haplotype (so 3\*20=60X for a triploid) and a heterozygosity level of at least 0.4% (average of 1 heterozygous SNP every 250 bp).

It is currently untested on pacbio data so if you have a pacbio dataset (with a known ground truth) please contact me (raise an issue on github or email me), especially if you have errors.

If you have a hybrid sample that has an acquired genomic copy which is genetically distant from the rest of the genome, nPhase will struggle to predict accurate results as the "distant" haplotype will make the other haplotypes look incredibly similar to each other. The solution is to separate the long reads based on their genetic distance to a reference genome. More details in this [PDF file](/files/HybridPolyploidConsiderations.pdf).


### Contact me

email: oabousaada@unistra.fr  
discord: Peaceful#6956
