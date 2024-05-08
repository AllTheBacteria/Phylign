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

