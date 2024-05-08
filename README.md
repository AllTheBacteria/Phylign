# Phylign – search for sequences in the AllTheBacteria v0.2 dataset

<p>
<a href="https://brinda.eu/mof">
    <img src="docs/logo_wbg.svg" align="left" style="width:100px;" />
</a>
Alignment to the ATB v0.2 dataset; can be done on a single laptop/server or a cluster.
Phylign uses
<a href="https://brinda.eu/mof">
phylogenetically compressed assemblies and their <i>k</i>-mer
indexes</a>
to align batches of queries to them by
<a href="https://github.com/lh3/minimap2">Minimap 2</a>.
</p><br/>

[![Info](https://img.shields.io/badge/Project-Info-blue)](https://brinda.eu/mof)
[![Paper DOI](https://img.shields.io/badge/paper-10.1101/2023.04.15.536996-14dc3d.svg)](https://doi.org/10.1101/2023.04.15.536996)
[![GitHub release](https://img.shields.io/github/release/karel-brinda/phylign.svg)](https://github.com/karel-brinda/phylign/releases/)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.10828248.svg)](https://doi.org/10.5281/zenodo.10828248)
[![CI Tests](https://github.com/karel-brinda/phylign/actions/workflows/ci.yaml/badge.svg)](https://github.com/karel-brinda/phylign/actions/)



## 1. Introduction

The central idea behind Phylign, enabling alignment locally at such a large
scale, is [**phylogenetic compression**](https://brinda.eu/mof)
([paper](https://doi.org/10.1101/2023.04.15.536996)). In short, input data are batched and reordered according to the topology of the estimated
phylogenies, which makes data highly locally compressible even using basic
techniques. Existing software packages for compression, indexing, and search
- in this case [XZ](https://tukaani.org/xz/),
[COBS](https://github.com/iqbal-lab-org/cobs), and
[Minimap2](https://github.com/lh3/minimap2) - are then used as low-level tools.


For more information about phylogenetic compression and the implementation
details of Phylign, see the [corresponding
paper](https://www.biorxiv.org/content/10.1101/2023.04.15.536996v2) (including
its [supplementary
material](https://www.biorxiv.org/content/biorxiv/early/2023/04/18/2023.04.15.536996/DC1/embed/media-1.pdf)
and visit the [associated website](https://brinda.eu/mof).


## 2. Requirements

### 2a) Hardware

Phylign requires a standard desktop or laptop computer with an \*nix system,
and it can also run on a cluster.

**WARNING: Phylign does not currently work on systems with ARM processors.**


### 2b) Dependencies

Phylign is implemented as a [Snakemake](https://snakemake.github.io)
pipeline, using the Conda system to manage non-standard dependencies. Ensure
you have [Conda](https://docs.conda.io/en/latest/miniconda.html) installed with
the following packages:

* [GNU Time](https://www.gnu.org/software/time/) (on Linux present by default; on OS X, install with `brew install gnu-time`).

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

For additional info see the additional info file.

## 6. License

[MIT](https://github.com/AllTheBacteria/Phylign/blob/main/LICENSE)


### Citation

> K. Břinda, L. Lima, S. Pignotti, N. Quinones-Olvera, K. Salikhov, R. Chikhi, G. Kucherov, Z. Iqbal, and M. Baym. **[Efficient and Robust Search of Microbial Genomes via Phylogenetic Compression.](https://doi.org/10.1101/2023.04.15.536996)** *bioRxiv* 2023.04.15.536996, 2023. https://doi.org/10.1101/2023.04.15.536996





## 7. Contacts
If you want to know about All The Bacteria, contact [Zamin Iqbal] \<zi245@bath.ac.uk\> or [John Lees] \<jlees@ebi.ac.uk\>.
If you want to know about Phylign on All the Bacteria, contact [Dan Anderson] \<dander@ebi.ac.uk\> or [Wei Shen] \<shenwei@gmail.com\> 
If you want to know about Phylign generallym contact [Karel Brinda](https://brinda.eu) \<karel.brinda@inria.fr\>.

