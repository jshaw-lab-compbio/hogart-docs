## Very quick start

```sh
# install + download ribosomal protein HMMs for filtering
conda install -c bioconda hogart 
wget "https://zenodo.org/records/23252787/files/bac120+rbp.hmm.gz?download=1" -O bac120+rbp.hmm.gz 
gzip -d bac120+rbp.hmm.gz
```

#### (Option 1) Input: Prokka / Bakta annotated genomes (fna + gff + faa)
```sh
hogart end-to-end \
  --annotated-genomes bakta_folder1 prokka_folder2 \
  -c bac120+rbp.hmm -t 30
```

#### (Option 2) Input: Raw fasta files

```sh
hogart end-to-end --genomes genomes/*.fa -c bac120+rbp.hmm -t 30
```

#### (Option 3) Input: GFF + Fasta + Protein

```sh
hogart end-to-end --gff gffs/* \
  --gff-fna fasta/* \
  --gff-faa proteins/* \
  -c bac120+rbp.hmm -t 30
```

## Install options

#### Conserved HMM database for filtering

We use a set of [profile HMMs](https://zenodo.org/records/23252787) for filtering spurious HGT (often ribosomal proteins). Download it using

```sh
wget "https://zenodo.org/records/23252787/files/bac120+rbp.hmm.gz?download=1" -O bac120+rbp.hmm.gz 
gzip -d bac120+rbp.hmm.gz
```



#### Option 1: conda install 
[![Anaconda-Server Badge](https://anaconda.org/bioconda/hogart/badges/version.svg)](https://anaconda.org/bioconda/hogart)
[![Anaconda-Server Badge](https://anaconda.org/bioconda/hogart/badges/latest_release_date.svg)](https://anaconda.org/bioconda/hogart)

```sh
conda install -c bioconda hogart
```

#### Option 2: Build from source

If you build from hogart from source, the following additional software needs to be in [PATH](https://en.wikipedia.org/wiki/PATH_(variable)):

1. (Required) [hmmer](https://hmmer.org/). Tested with v3.4. Required unless `--skip-hmms` is specified. 
2. (Optional) [pyrodigal](https://pyrodigal.readthedocs.io/en/stable/). Only required if you use raw fasta (without annotation). Tested with v3.6.3
3. (Very optional) [graphviz](https://graphviz.org/). Only needed for drawing module pictures with `--write-graphs`.

To build hogart, we require a rust toolchain:

1. [rust](https://www.rust-lang.org/tools/install) 
2. A c compiler (e.g. GCC)
3. make

Building takes a few minutes (depending on # of cores).

```sh
git clone https://github.com/bluenote-1577/hogart
cd hogart 
cargo install --path . 
```


