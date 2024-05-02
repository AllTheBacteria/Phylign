# Phylign – search for sequences in the AllTheBacteria v0.2 dataset

<p>
<a href="https://brinda.eu/mof">
    <img src="docs/logo_wbg.svg" align="left" style="width:100px;" />
</a>
Alignment to
<a href="https://doi.org/10.1371/journal.pbio.3001421">all pre-2019 bacteria from ENA</a>
on standard desktop and laptops computers.
Phylign uses
<a href="https://brinda.eu/mof">
phylogenetically compressed assemblies and their <i>k</i>-mer
indexes</a>
to align batches of queries to them by
<a href="https://github.com/lh3/minimap2">Minimap 2</a>,
all within only several hours.
</p><br/>

[![Info](https://img.shields.io/badge/Project-Info-blue)](https://brinda.eu/mof)
[![Paper DOI](https://img.shields.io/badge/paper-10.1101/2023.04.15.536996-14dc3d.svg)](https://doi.org/10.1101/2023.04.15.536996)
[![GitHub release](https://img.shields.io/github/release/karel-brinda/phylign.svg)](https://github.com/karel-brinda/phylign/releases/)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.10828248.svg)](https://doi.org/10.5281/zenodo.10828248)
[![CI Tests](https://github.com/karel-brinda/phylign/actions/workflows/ci.yaml/badge.svg)](https://github.com/karel-brinda/phylign/actions/)

<h2>Contents</h2>

<!-- vim-markdown-toc GFM -->

* [1. Introduction](#1-introduction)
  * [Citation](#citation)
* [2. Requirements](#2-requirements)
  * [2a) Hardware](#2a-hardware)
  * [2b) Dependencies](#2b-dependencies)
* [3. Installation](#3-installation)
  * [3a) Step 1: Install dependencies](#3a-step-1-install-dependencies)
  * [3b) Step 2: Clone the repository](#3b-step-2-clone-the-repository)
* [4. Usage](#4-usage)
  * [4a) Step 1: Copy or symlink the assemblies](#4a-step-1-copy-or-symlink-the-assemblies)
  * [4b) Step 2: Copy or symlink the compressed COBS indices](#4b-step-1-copy-or-symlink-the-compressed-cobs-indices)
  * [4c) Step 3: Copy or symlink your queries](#4c-step-1-copy-or-symlink-your-queries)
  * [4d) Step 4: Adjust configuration](#4d-step-2-adjust-configuration)
  * [4e) Step 5: Clean up intermediate files](#4e-step-3-clean-up-intermediate-files)
  * [4f) Step 6: Run the pipeline](#4f-step-4-run-the-pipeline)
  * [4g) Step 7: Analyze your results](#4g-step-5-analyze-your-results)
* [5. Additional information](#5-additional-information)
  * [5a) List of workflow commands](#5a-list-of-workflow-commands)
  * [5b) Directories](#5b-directories)
  * [5c) Querying a subset of the AllTheBacteria dataset](#5c-querying-a-subset-of-the-allthebacteria-dataset)
  * [5d) File formats](#5d-file-formats)
  * [5e) Running on a cluster](#5e-running-on-a-cluster)
  * [5f) Known limitations](#5f-known-limitations)
* [6. License](#6-license)
* [7. Contacts](#7-contacts)

<!-- vim-markdown-toc -->


## 1. Introduction

The central idea behind Phylign, enabling alignment locally at such a large
scale, is [**phylogenetic compression**](https://brinda.eu/mof)
([paper](https://doi.org/10.1101/2023.04.15.536996)) - a technique based on
using estimated evolutionary history to guide compression and search of large
genome collections using existing algorithms and data structures.

In short, input data are reorganized according to the topology of the estimated
phylogenies, which makes data highly locally compressible even using basic
techniques. Existing software packages for compression, indexing, and search
- in this case [XZ](https://tukaani.org/xz/),
[COBS](https://github.com/iqbal-lab-org/cobs), and
[Minimap2](https://github.com/lh3/minimap2) - are then used as low-level tools.
The resulting performance gains come from a wide range of benefits of
phylogenetic compression, including easy parallelization, small memory
requirements, small database size, better memory locality, and better branch
prediction.

For more information about phylogenetic compression and the implementation
details of Phylign, see the [corresponding
paper](https://www.biorxiv.org/content/10.1101/2023.04.15.536996v2) (including
its [supplementary
material](https://www.biorxiv.org/content/biorxiv/early/2023/04/18/2023.04.15.536996/DC1/embed/media-1.pdf)
and visit the [associated website](https://brinda.eu/mof).


### Citation

> K. Břinda, L. Lima, S. Pignotti, N. Quinones-Olvera, K. Salikhov, R. Chikhi, G. Kucherov, Z. Iqbal, and M. Baym. **[Efficient and Robust Search of Microbial Genomes via Phylogenetic Compression.](https://doi.org/10.1101/2023.04.15.536996)** *bioRxiv* 2023.04.15.536996, 2023. https://doi.org/10.1101/2023.04.15.536996


## 2. Requirements

### 2a) Hardware

Phylign requires a standard desktop or laptop computer with an \*nix system,
and it can also run on a cluster. The minimal hardware requirements are **12 GB
RAM** and approximately **120 GB of disk space** (102 GB for the database and
a margin for intermediate files).

**WARNING: Phylign does not currently work on devices with ARM processors.**


### 2b) Dependencies

Phylign is implemented as a [Snakemake](https://snakemake.github.io)
pipeline, using the Conda system to manage non-standard dependencies. Ensure
you have [Conda](https://docs.conda.io/en/latest/miniconda.html) installed with
the following packages:

* [GNU Time](https://www.gnu.org/software/time/) (on Linux present by default; on OS X, install with `brew install gnu-time`).
* [Python](https://www.python.org/) (>=3.7)
* [Snakemake](https://snakemake.github.io) (>=6.2.0)
* [Mamba](https://mamba.readthedocs.io/) (>= 0.20.0) - optional, but recommended

Additionally, Phylign uses standard Unix tools like
[GNU Make](https://www.gnu.org/software/make/),
[cURL](https://curl.se/),
[XZ Utils](https://tukaani.org/xz/), and
[GNU Gzip](https://www.gnu.org/software/gzip/).
These tools are typically included in standard \*nix installations. However, in
minimal setups (e.g., virtualization, continuous integration), you might need
to install them using the corresponding package managers.


## 3. Installation

### 3a) Step 1: Install dependencies

Make sure you have Conda and GNU Time installed. On Linux:

```bash
sudo apt-get install conda
```

On OS X (using Homebrew):

```bash
brew install conda
brew install gnu-time
```

### 3b) Step 2: Clone the repository

Clone the Phylign repository from GitHub and navigate into the directory:

```bash
 git clone https://github.com/AllTheBacteria/Phylign
 cd phylign
```

### 3c) Step 3: Install dependencies

```bash
conda env create -f environment.yaml && conda activate phylign
```

## 4. Usage

### 4a) Step 1: Copy or symlink the assemblies

Copy or symlink the miniphy-compressed batches of assemblies you want to map to and place them in `asms/`.

### 4b) Step 2: Copy or symlink the compressed COBS indices

Copy or symlink the miniphy-cobs compressed batches of search indices you want to query and place them in `cobs/`. Each batch in `asms/` should have a matching index in `cobs/`.

### 4c) Step 3: Copy or symlink your queries

Remove the default test files or your old files in the `input/` directory and
copy or symlink (recommended) your query files. The supported input formats are
FASTA and FASTQ, possibly gzipped. All query files will be preprocessed and
merged together.

*Notes:*
* All query names have to be unique among all query files.
* Queries should not contain non-ACGT characters. All non-`ACGT` characters in your query sequences will be translated to `A`.


### 4d) Step 4: Adjust configuration

Edit the [`config.yaml`](config.yaml) file for your desired search. All
available options are documented directly there.

### 4e) Step 5: Clean up intermediate files

Run `make clean` to clean intermediate files from the previous runs. This
includes COBS matching files, alignment files, and various reports.

### 4f) Step 6: Run the pipeline

Simply run `make`, which will execute Snakemake with the corresponding
parameters. If you want to run the pipeline step by step, run `make match`
followed by `make map`.

### 4g) Step 7: Analyze your results

Check the output files in `output/` (for more info about formats, see
[5c) File formats](#5c-file-formats)).

If the results do not correspond to what you expected and you need to re-adjust
your search parameters, go to Step 2. If only the mapping part is affected by
the changes, you proceed more rapidly by manually removing the files in
`intermediate/05_map` and `output/` and running directly `make map`.


## 5. Additional information

### 5a) List of workflow commands

Phylign is executed via [GNU Make](https://www.gnu.org/software/make/),
which handles all parameters and passes them to Snakemake.

Here's a list of all implemented commands (to be executed as `make {command}`):


```bash
####################
# General commands #
####################
   all                Run everything (the default rule)
   help               Print help messages
   clean              Clean intermediate search files
   cleanall           Clean all generated and downloaded files
##################
# Pipeline steps #
##################
   conda              Create the conda environments
   match              Match queries using COBS (queries -> candidates)
   map                Map candidates to assemblies (candidates -> alignments)
#############
# Reporting #
#############
   config             Print configuration without comments
   report             Generate Snakemake report
###########
# Cluster #
###########
   cluster_slurm      Submit to a SLURM cluster
   cluster_lsf        Submit to LSF cluster
##################
# For developers #
##################
   format             Reformat Python and Snakemake files
   checkformat        Check source code format
```

*Note:* `make format` requires
[YAPF](https://github.com/google/yapf) and
[Snakefmt](https://github.com/snakemake/snakefmt), which can be installed by
`conda install -c conda-forge -c bioconda yapf snakefmt`.

### 5b) Directories

* `asms/`, `cobs/` Downloaded assemblies and COBS indexes
* `input/` Queries, to be provided within one or more FASTA/FASTQ files,
  possibly gzipped (`.fa`)
* `intermediate/` Intermediate files
   * `00_queries_preprocessed/` Preprocessed queries
   * `01_queries_merged/` Merged queries
   * `02_cobs_decompressed/` Decompressed COBS indexes (temporary, used only in
     the disk mode is used)
   * `03_match/` COBS matches
   * `04_filter/` Filtered candidates
   * `05_map/` Minimap2 alignments
* `logs/` Logs and benchmarks
* `output/` The resulting files (in a headerless SAM format)

### 5c) Querying a subset of the AllTheBacteria dataset

It is possible to run Phylign on a subset of the AllTheBacteria assemblies if e.g. you only want to query a certain species or your resources are limited. This can be done by downloading the desired assemblies and COBS indices and following the steps described in [Usage](#4-usage). You then need to modify `data/batches_2m.txt` to only include batches you have assemblies and compressed COBS indices for. E.g. to search only `asms/salmonella_enterica__81.asm.tar.xz` using the compressed index `cobs/salmonella_enterica__81.cobs_classic.xz`, you must modify the file to only include `salmonella_enterica__81`. Alternatively, you can create a new `.txt` file with one batch per line, and set the `batches` variable in `config.yaml` to the path of this new file.

### 5d) File formats

**Input files:** FASTA or FASTQ files possibly compressed by gzipped. The files
are searched in the `input/` directory, as files with the following suffixes:
`.fa`, `.fasta`, `.fq`, `.fastq` (possibly with `.gz` at the end).

**Output files:**

* `output/{name}.sam_summary.gz`: output alignments in a headerless SAM format
* `output/{name}.sam_summary.stats`: statistics about your computed alignments
  in TSV

SAM headers are omitted as all search experiments
generate hits across large numbers of assemblies (many
of them being spurious). As a result, SAM headers then
dominate the outputs. Nevertheless, we note that, in
principle, the SAM headers can always be recreated from the
FASTA files in `asms/`, although this functionality is not
currently implemented.


### 5e) Running on a cluster

Running on a cluster is much faster as the jobs produced by this pipeline are
quite light and usually start running as soon as they are scheduled.

**For LSF clusters:**

1. Setup the snakemake LSF profile described [here](https://github.com/Snakemake-Profiles/lsf).
2. Configure you queries and run the full pipeline: `make cluster_lsf`;

**For SLURM clusters:**
1. Setup the snakemake LSF profile described [here](https://github.com/Snakemake-Profiles/slurm).
2. Configure you queries and run the full pipeline: `make cluster_slurm`;

### 5f) Known limitations

* **Swapping if the number of queries too high.** If the number of queries is
  too high (e.g., 10M Illumina reads), the auxiliary Python scripts start to
  use too much memory, which may result in swapping. Try to keep the number of
  queries moderate and ideally their names short.
* **No support for ambiguous characters in queries.** Queries are expected to
  be over the ACGT alphabet. All non-ACGT characters in queries are first
  converted to A.
* **Too many reported hits.** When queries have too many equally good hits in
  the database, even if the threshold on the maximum number of hits is chosen
  low – for instance 10 – the program will take top 10 + ties, which can be
  a huge number (especially for short sequences).


## 6. License

[MIT](https://github.com/karel-brinda/phylign/blob/master/LICENSE)



## 7. Contacts

* [Karel Brinda](https://brinda.eu) \<karel.brinda@inria.fr\>
* [Leandro Lima](https://github.com/leoisl) \<leandro@ebi.ac.uk\>
