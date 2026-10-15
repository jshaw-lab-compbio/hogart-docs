
## Output folder layout

```
hogart_results/
├── module_overview.yaml    ← start here
├── module_loci.tsv
├── module_loci.bed
├── module_details.json     (very verbose, but complete)
├── prepare/                (gene calls)
├── run/                    (intermediate files)
└── extract/
    └── modules/            (one folder per module)
        ├── module_7/
        ├── module_12_var1/
        └── module_12_var2/
```

The four files in the top folder are shortcuts to files in `extract/`. 

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
    genes:
      - {"locus_tag": "MALGFC_04114", "gene_name": "traG", "product": "Conjugative transfer protein", "dbxref": null}
      - {"locus_tag": "MALGFC_04115", "gene_name": null, "product": "Hypothetical protein", "dbxref": null}
```

**`module_loci.tsv` - where each module is found.** One row per locus (a module integrated in a genome). A genome can have more than one copy of a module.

**`module_loci.bed` - the same loci for genome browsers** (IGV, bedtools, etc.). BED files only store contig names, so if two genomes share contig names, use the TSV.

**`module_details.json` - everything, for scripting.** Includes the above plus actual gene sequences and genomic coordinates. 

## Per-module folders

Each module gets a folder in `extract/modules/` containing:

| File | Contents |
|---|---|
| `genes.fna` / `genes.faa` | The module's genes (nucleotide / protein), in order |
| `source_path.json` | Summary file of the present genes and the consensus step (if a non-variant module) |
| `loci/*.gbk` | GenBank format files for each loci. Useful for visualization with [clinker](https://github.com/gamcil/clinker) |
| `gene_families/col*.fna` | (requires `--write-gene-families`) The original gene sequences for each consensus gene, prior to collapsing. |

Each gene in `genes.fna` / `genes.faa` has a header with its locus tag, gene name, product, and **frequency**: the fraction of loci carrying that gene, e.g. `freq=0.875|n=7/8` means 7 of 8 loci. Frequencies are weighted so that near-identical genomes do not dominate.

## Module variants

Mobile elements are often mosaic: the same element may carry different cargo genes in different genomes. When hogart finds several distinct gene-content versions of a module, it reports them as **variants** (`module_12_var1`, `module_12_var2`, ...) instead of one combined module. The variant numbers are arbitrary labels.

## Interpreting modules

- A module is a **candidate** horizontally transferred region. Look at its genes, its loci, and its biological context before calling it an HGT event.
- Module boundaries are approximate. Compare the loci (e.g., `loci/*.gbk` in clinker) to check genes near the ends.
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
