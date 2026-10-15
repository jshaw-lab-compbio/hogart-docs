
## Output folder layout

```
hogart_results/
├── module_overview.yaml    ← start here
├── module_loci.tsv
├── module_loci.bed
├── module_details.json     (very verbose, but complete)
└── modules/                (shortcut to extract/modules/)
    |── module_7/
    ├── module_12_var1/
    └── module_12_var2/
├── hogart.log
├── prepare/                (gene calls)
├── run/                    (intermediate files)
```

The four summary files and `modules/` in the top folder are shortcuts to files in `extract/`. If `run` finds no candidate paths, `extract` is skipped and these shortcuts are not created.

## Summary files

**`module_overview.yaml` - a human-readable list of every module.** For each module: how many genomes and loci it was found in, its average length, and its genes in order. A genome can have multiple loci. 

```yaml
modules:
  - module_id: "module_433_var2"
    parent_module: "module_433"
    num_genomes: 7
    num_loci: 9
    num_genes: 18
    mean_length_bp: 22450.0
    directory: "/absolute/path/hogart_results/extract/modules/module_433_var2"
    genes:
      - {"locus_tag": "MALGFC_04114", "gene_name": "traG", "product": "Conjugative transfer protein", "dbxref": null, "support": "8/9"}
      - {"locus_tag": "MALGFC_04115", "gene_name": null, "product": "Hypothetical protein", "dbxref": null, "support": "7/9"}
```

`support: "50/60"` means 50 of the 60 regions assigned to this module or variant contain that gene. These are raw counts, but are not yet weighted by diversity. 

**`module_loci.tsv` - where each module is found.** One row per locus (a module integrated in a genome). A genome can have more than one copy of a module.

Note: loci with fewer than two distinct retained consensus genes are omitted. 

**`module_loci.bed` - the same loci for genome browsers** (IGV, bedtools, etc.). This is BED4: contig, zero-based start, exclusive end, and module ID. BED files only store contig names, so if two genomes share contig names, use the TSV.

**`module_details.json` - complete module summaries, for scripting.** Includes gene annotations, raw and weighted support, final genomic coordinates, length statistics, source-path provenance, and absolute paths to the output files. 

## Per-module folders

Each module gets a folder in `extract/modules/` containing:

| File | Contents |
|---|---|
| `genes.fna` / `genes.faa` | The module's genes (nucleotide / protein), in order |
| `module.json` | This module's record from `module_details.json` |
| `loci.tsv` | One row per final locus, with coordinates, gene count, and its GenBank filename |
| `source_path.json` | The original padded regions and their gene sequences, before consensus extraction (non-variant modules only; see below for variants) |
| `loci/*.gbk` | GenBank files for observed loci, written by default unless `--no-genbank` is used. Useful for visualization with [clinker](https://github.com/gamcil/clinker) |
| `gene_families/col*.fna` / `col*.faa` | (requires `--write-gene-families`) All mapped nucleotide / protein copies for each retained consensus gene, not just unique sequences |

Example records from a real `genes.faa`:

```fasta
>OCPOJN_01061|-|freq=0.676|n=4/5|col68 UPF0237 protein SSA_2019 dbxref=SO:0001217,UniRef:UniRef50_A3CQD4
MKAIITVVGKDQKGIVAGVATKVAELGLNIDDISQTVLDEYFTMMAVVSSDEKKDFTQLRSELEEFGQSLHVKINIQSAAIFDAMHNL
>OCPOJN_01072|-|freq=0.486|n=2/5|col98 hypothetical protein dbxref=SO:0001217,UniRef:UniRef50_K8MUX3
MNKYRVIYYLCIIAFIANVSMMIGTLLGWFHILDSNYFFWSSLILSLVFERIEKKVKERV
```

Each gene in `genes.fna` / `genes.faa` has a header such as `LOCUS_TAG|gene_name|freq=0.875|n=7/8|col17 product`. 

- `n=7/8` is raw region support, using the same basis as YAML `support`. 
- Frequency is the diversity-weighted fraction of assigned-region weight carrying the gene, so it need not equal 7/8. Near-identical genomes are downweighted. 

GenBank files contain translated genes observed within each final locus in their original order and orientation. Intergenic or unavailable DNA is filled with Ns. 

## Module variants

Mobile elements are often mosaic: the same element may carry different cargo genes in different genomes. When hogart finds several distinct gene-content versions of a module, it reports them as **variants** (`module_12_var1`, `module_12_var2`, ...) instead of one combined module. The variant numbers are arbitrary labels.

The combined consensus of variant modules can be inspected under `extract/debug_modules/module_12/`.

## Interpreting modules

- A module is a **candidate** horizontally transferred region. Look at its genes, its loci, and its biological context before calling it an HGT event.
- Module boundaries are approximate and are limited to **gene boundaries**. Compare the loci (e.g., `loci/*.gbk` in clinker) to check genes near the ends.
- Module IDs change between runs. To compare runs, compare loci, not IDs.

## Outputs of each step (`prepare`, `run`, `extract`)

`end-to-end` runs three steps, each with its own folder. Each folder has a `hogart.log` with progress and settings; check it first if something looks wrong (e.g., no modules found).

**`prepare/`: gene calls.** Genes and proteins for each genome, plus `manifest.tsv`, which lists the files for every genome.

**`run/`: candidate modules.** The main search step.

| File | Contents |
|---|---|
| `clusters.tsv` | Clusters of near-identical genes found in different genomes |
| `paths.tsv` | Raw candidate modules and the genes in each, before cleaning |
| `path_regions.json` | Candidate regions with 25 kb of flanking genes on each side; the input to `extract` |

**`extract/`: final modules.** Cleans up the candidates and finds variants. All the files described above (summary files and `modules/`) are here.

