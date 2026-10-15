## Overview

Hogart runs in three steps:

1. **`prepare`**: collects genes and proteins for each genome (calling genes first if needed).
2. **`run`**: finds candidate modules shared between genomes. This is the main step.
3. **`extract`**: cleans up the candidates into final modules and finds module variants.

**`hogart end-to-end` runs all three steps with one command.** This is what most users want:

```sh
hogart end-to-end --annotated-genomes bakta_or_prokka_genomes/*\
     -c rbp.hmm -t 30 --out-dir hogart_results
```

### Common options

| Option | Meaning |
|---|---|
| `--annotated-genomes` / `--genomes`| See [input options below](#inputs) |
| `--out-dir` | Output folder |
| `-t` | Number of threads |
| `-c FILE.hmm` | HMM file of conserved genes (e.g., ribosomal proteins) to filter out. Can be given multiple times |
| `--skip-hmms` | Don't filter conserved genes. Either `-c` or `--skip-hmms` is required |

Run `hogart end-to-end --help` for all options.

## Inputs

Hogart takes a set of genomes (isolates or MAGs), **one genome per file**. A genome file can contain multiple contigs. Hogart finds modules shared **between** genomes, never within one.

Hogart needs a genome FASTA, a GFF, and protein sequences for every genome. Gene annotations (gene names, products) are not used by the algorithm; they only make the output easier to read. All input files can be gzipped.

### Input options

**1. Raw FASTA files (no annotation).** Hogart calls genes for you with [pyrodigal](https://pyrodigal.readthedocs.io/en/stable/).

```sh
hogart end-to-end --genomes genomes/*.fa -c rbp.hmm -t 30
```

**2. Annotation folders (Bakta, Prokka, or NCBI RefSeq downloads).** Point hogart at one folder per genome.

```sh
hogart end-to-end --annotated-genomes bakta/genome1 bakta/genome2 -c rbp.hmm -t 30
```

Each folder must contain a genome FASTA (`.fna`), a GFF (`.gff` or `.gff3`), and proteins (`.faa`). A `.ffn` file is used if present but is not required.

**3. Your own GFF + FASTA + protein files.** Give the three files for each genome **in the same order**:

```sh
hogart end-to-end \
  --gff gffs/g1.gff gffs/g2.gff \
  --gff-fna fasta/g1.fna fasta/g2.fna \
  --gff-faa proteins/g1.faa proteins/g2.faa \
  -c rbp.hmm -t 30
```

- Protein FASTA headers must match the CDS IDs in the GFF.
- Genome names are taken from the folder containing each GFF (handy for NCBI downloads where every file is called `genomic.gff`). Add `--genome-id-stem` to name genomes by the GFF filename instead.

!!! note "Protein sequences are required"
    Hogart stops with an error if a genome has no matching proteins. Genes without a protein (e.g., pseudogenes) are silently skipped.

### Treating contigs as genomes (metagenome assemblies)

You can also treat every contig in an assembly as its own genome, e.g., to find modules shared between organisms in a single metagenome. Use `--contigs` (raw FASTA) or `--annotated-contigs` (annotation folder):

```sh
hogart end-to-end --contigs assembly.fa --meta -c rbp.hmm -t 30
```

!!! warning "Use long or complete contigs"
    A module can only be found if it sits on a contig with its genomic context. This mode works best with long, ideally complete, contigs (e.g., from long-read assemblies). Short, fragmented contigs from short-read assemblies will give poor results.

- `--meta` runs pyrodigal in metagenome mode, which is recommended for assemblies.
- Contigs shorter than 3 kb are ignored by default. We recommend raising this a lot with `--min-contig-len`.

## Running the steps separately

`end-to-end` is equivalent to running these three commands:

```sh
hogart prepare --genomes genomes/*.fa --out-dir prepare -t 30
hogart run prepare/manifest.tsv --out-dir run -c rbp.hmm -t 30
hogart extract --input run/path_regions.json --out-dir extract -t 30
```

`prepare` accepts the same input options as `end-to-end` (see [Inputs](#inputs)).

### Rerunning `extract` with different settings

`extract` is fast compared to `run`. To change how modules are cleaned up or which extra files are written, you can rerun `extract` on the `run/` output instead of rerunning everything:

```sh
hogart extract \
  --input hogart_results/run/path_regions.json \
  --out-dir extract_v2 \
  --write-gene-families \
  -t 30
```

Useful options:

- `--min-frequency` (default 0.65): how common a gene must be across loci to be kept at the ends of a module. Lower it to get longer modules that include rarer genes; raise it to get stricter, shorter modules.
- `--write-gene-families`: write every version of each module gene (see [Output formats](outputs.md#per-module-folders)).
- `--path-id N`: only process candidate `N` (the ID in `run/paths.tsv`). Repeat to select several.

!!! tip
    Keep `path_regions.json` in the `run/` folder; `extract` also uses the `clusters.tsv` next to it. Use a new `--out-dir` so your original results are not overwritten. The shortcuts in `hogart_results/` still point to the original `extract/`.
