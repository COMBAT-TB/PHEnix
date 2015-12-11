# Phoenix [![build status](http://bioinformatics-git.phe.gov.uk/ci/projects/1/status.png?ref=master)](http://bioinformatics-git.phe.gov.uk/ci/projects/1?ref=master)

If you have a sample and you want to have one-stop-shop analysis run the following:

```
run_snp_pipeline.py -r1 <path to R1.fastq> -r2 <path to R2.fastq> \ 
-r <path to reference> --sample-name <NAME> --mapper bwa --variant gatk \ 
--filters min_depth:5,mq_score:30
```

This will map with **BWA** and call variants with **GATK**. Intermediate files are written into the same directory you run this 
command from. **--sample-name** option is very important, it specifies what output files will be called and the read group in the BAM
file. If you omit it, **test_sample** will be used.

## Filters

One of the key parts of the VCF processing is to filter quality calls. To do this we have created a flexible interface with
variaety of filters avaialble:

- **qual_score** - Filter records where **QUAL** score is below given threshold.
- **ad_ratio** - Filter, defined by **gatk**, records where ratio of alt allele to sum of all alleles is below given fraction.
- **dp4_ratio** - Similar to **ad_ratio**, but used in **mpileup** variant caller.
- **mq_score** - Filter records that fall below specified **MQ** score (from _INFO_ field).
- **mq0_ratio** - Filter, defined by **gatk**, records with **MQ0** to **DP** ratio _above_ given threshold (both values are from _INFO_ field).
- **mq0f_ratio** - Similar to the **mq0_ratio**, but used in **mpileup** variant caller.
- **qg_score** - Filter records that fall below specified **GQ** score (from **first** sample).
- **min_depth** - Filter records with mean depth below specified threshold (**DP** from sample field).
- **uncall_gt** - Filter records with uncallable genotypes in VCF (**GT** is ./.).

All filters are applied for each position and those positions that pass ALL filters are kept as quality calls. Positions
failing filter will be kept for future reference and creating fasta files, when needed. To specify filters to be used, simply
list them as key:threshold pairs separated by comma(,). For filters that don't require threshold, leave blank after ':'. 

## Annotators

Individual VCFs can be annotated using custom annotators. Currently available annotators:

- **coverage** - Annotates with information about _mean_ and _dev_ of the depth in the VCF (using DP from INFO).

The data can be accessed from the metadata field in the VCF in the following way:

```python
r = vcf.Reader(filename="/path/to/vcf")
print r.metadata["coverageMetaData"][0]["mean"]
print r.metadata["coverageMetaData"][0]["dev"]
```

## Converting from VCF to FASTA

A lot of downstream applications take on FASTA formated files, not VCFs. We have included a script for converting VCF data to
FASTA format.

```
vcfs2fasta -d <path to directory of VCFs> -o <path to output FASTA>
```

This tool is also able to filter out samples and columns that may be bad quality. E.g. **--sample-Ns** specifies maximum fraction of Ns
present in a sample. **--Ns** specifies maximum fraction of Ns allowed per column. **--with-mixtures** can specify if mixed position 
should be output as over certain fraction. First, samples are sifted out then columns are checked.

## Requirements

A lot of functionality depends on the presence of existing 3rd party tools:

Mappers:

- BWA
- Bowtie2

Variant Caller:

- GATK
- MPileup

In order for them to function properly, they need to be already in you *PATH*. For commands that
run through Java archives, please set appropriate environment variable (see below).

Python:

- Python >= 2.7
- argparse
- PyVCF
- PyYAML
- matplotlib (_optional_)
- bintrees (_optional_)
- numpy (_optional_)
- matplotlib.venn (_optional_)
- psycopg2 (_optional_)

## 3rd Party Requirements

### Samtools

Samtools can be downloadded from blurb. It is used to filter and convert to SAM/BAM files and in mpileup variant caller.

### BCFTools

BCFtools can be downloaded from blurb. It is used for calling variants in mpileup.

### BWA

Heng Li's mapper can be downloaded from blurb.

### Bowtie2

Bowtie2 mapper available from blurb.

### GATK
Set *GATK_JAR* - full path to the GATK Java archive.

### Picard
Picard is needed for GATK to create dictionary of reference fasta.
Either set *PICARD_TOOLS_PATH* - path to directory where different Picard jars are or set *PICARD_JAR* - path to **picard.jar**.
Older Picard distributions have many different jars (use first suggestion), where as newer versions have merged all into one jar file. 



## History

In the beginning there were 2 pipelines to call SNPs: upstairs and downstairs.
One day we decided it was a bad idea to have 2 pieplines that did similar things.
The existing pipelines died, quiet software death, and new pipeline was born, to rule
them all. Just like the ring.