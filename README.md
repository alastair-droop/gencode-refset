# GENCODE Reference Set Generation

## Overview

Multiple different reference genomes are required to run different alignment tools. However, many of these share the same input reference datasets. In order to prevent downloading multiple copies of reference data, we can split the indices into the raw data (`.fasta` and `.gtf`) files (*reference sets*), and the aligner-specific indices (*indices*).

This set of scripts automates the process of downloading reference sets from [GENCODE](https://www.gencodegenes.org).

These scripts are run through the [qsubsec](https://github.com/alastair-droop/qsubsec) template engine.

## Reference Set Types

Currently, four different types of reference set types are specified:

Type | Label | Description
:----|:----:|:----
`genome` | **G** | The complete genome organism sequence (including reference chromosomes, scaffolds, assembly patches and haplotypes)
`transcript` | **T** | The nucleotide sequences for all transcripts on the reference chromosomes
`pc` | **P** | The nucleotide sequences for all protein-coding transcripts on the reference chromosomes (biotypes `protein_coding`, `nonsense_mediated_decay`, `non_stop_decay`, `IG_*_gene`, `TR_*_gene`, `polymorphic_pseudogene`)
`lnc` | **L** | The nucleotide sequences for all long non-coding transcripts on the reference chromosomes

## Reference Set Identifiers

The reference set identifier is in the following format `{ACCESSION}p{PATCH}_{RELEASE}{LABEL}`, where:

* `ACCESSION` is the GENCODE genome accession;
* `PATCH` is the genome patch number;
* `RELEASE` is the GENCODE release number; and
* `LABEL` is the reference set type label (see above).

## Downloading a Specific Reference Set

The `gencode-refset.qsubsec` template downloads and indexes reference sets. The user must specify a base directory for the reference set. A directory within the base will be created for the new reference set. The genome accession, patch, release and type should also be specified.

The template files are hosted on [github](https://alastair-droop.github.io/gencode-refset/):

* `gencode-refset.qsubsec` at <https://alastair-droop.github.io/gencode-refset/gencode-refset.qsubsec>
* `gencode-refset.tff` at <https://alastair-droop.github.io/gencode-refset/gencode-refset.tff>

For example, to download the long non-coding reference set for Human GRCh38.12 data annotated with GENCODE release 29 to the reference set directory specified in the environment variable `$REFSET_DIR`:

~~~bash
qsubsec \
    https://alastair-droop.github.io/gencode-refset/gencode-refset.qsubsec \
    https://alastair-droop.github.io/gencode-refset/gencode-refset.tff \
    GENCODE_SPECIES=human \
    GENCODE_ACCESSION=GRCh38 \
    GENCODE_PATCH=12 \
    GENCODE_RELEASE=29 \
    REFSET_TYPE=lnc \
    REFSET_BASE=$REFSET_DIR
~~~

**NB**: When passing environment variables to qsubsec, make sure that they have been exported, otherwise they will not work.

## Template Tokens

The following tokens are necessary for running `gencode-refset.qsubsec`. The specified default values for all tokens (apart from `REFSET_BASE`) are specified in the `gencode-refset.tff` file.

Token | Default value | Description
:-----|:--------------|:-----------
`REFSET_BASE` | (no default) | The base directory for storing reference sets
`MAX_TIME` | `00:30:00` | The maximum job time when running on HPC
`MAX_MEM` | `1G` | The maximum job memory when running on HPC
`GENCODE_SPECIES` | `human` | The reference species (`human` or `mouse`)
`REFSET_TYPE` | `genome` | The reference type to create (see above)
`GENCODE_ACCESSION` | `GRCh38` | The genome accession to use
`GENCODE_PATCH` | `12` | The genome patch number to use
`GENCODE_RELEASE` | `29` | The GENCODE annotation release to use
`URL_BASE` | (see below) | The GENCODE base FTP location to use
`WGET_EXEC` | `wget` | The wget executable to use
`GZIP_EXEC` | `gzip` | The gzip executable to use
`SAMTOOLS_EXEC` | `samtools` | The samtools executable to use

The default FTP URL is set from the values of the other tokens, and is by default:
`ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_{GENCODE_SPECIES}/release_{GENCODE_RELEASE}`

## Reference Set Data

After download, the reference set directory will contain the appropriate `.fasta` and `.gtf` files for the reference type specified. The samtools `.fasta` file index is also included. The `logs` subdirectory contains the section logs, as well as a record of the GENCODE FTP locations used for the two reference files.

## Building Specific Index Sets

Template scripts for building aligner-specific indices are provided. For all of these to work, the following tokens must be supplied (as there are no specified defaults):

Token | Description
:-----|:-----------
`REFSET_BASE` | The base directory for storing reference sets
`INDEX_BASE` | The base directory for storing index sets
`REF_ID` | The reference set ID to build an index for

The following templates are provided:

Aligner  | Template File              | Token File
:--------|:---------------------------|:---------------------
`salmon` | `salmon-reference.qsubsec` | `salmon-reference.tff`
`STAR`   | `star-reference.qsubsec`   | `star-reference.tff`

The template and token files are hosted at <https://alastair-droop.github.io/gencode-refset/>. Aligner-specific tokens are usually required. Basic documentation for these tokens are given in the relevant `.tff` files.

For example, to build a Salmon reference for the reference set ID `GRCh38p12_29T` using the reference set base directory `$REFSET_DIR` and the index base directory `$REFINDEX_BASE`:

~~~bash
qsubsec \
    https://alastair-droop.github.io/gencode-refset/salmon-reference.qsubsec \
    https://alastair-droop.github.io/gencode-refset/salmon-reference.tff \
    REFSET_BASE=$REFSET_DIR \
    INDEX_BASE=$REFINDEX_DIR \
    REF_ID=GRCh38p12_29T
~~~
