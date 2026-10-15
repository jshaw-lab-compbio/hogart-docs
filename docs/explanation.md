### Common use cases:

You may want to use hogart to study:

- HGT between a diverse set of species (e.g., all species in a genera or family). 
- HGT across assembled contigs / MAGs in **a single metagenome sample**
    - Works best for long-read metagenome assemblies with high contiguity; modules are poorly assembled by short reads. 
- High quality MAGs from **different metagenomic samples**.
    - May work better for short reads, but be very careful of MAG contamination!

### Limitations of hogart:

- **Single-species datasets**. Hogart excels at finding HGT when the input is a **diverse set of species**. Hogart will not work if you dataset consists of only 1 species. 
    - Prefer pangenome methods like [PanRGP](https://microscope.readthedocs.io/en/stable/content/compgenomics/panRGP.html) if you want to study 1 species. 
- **Lack of HGT in dataset**. An HGT module must be shared across different species *in your dataset*. E.g., a prophage present in only 1 genome in your data will not be detected as a module. 
    - If you just want to detect putative HGT clusters in a single genome, consider genomic island (GI) prediction tools. 
- **Diverged HGT modules**. Hogart excels at finding **recently shared modules** (high sequence similarity). If two modules are shared at < 99% identity, they are considered different modules. 

### How does hogart work?

Hogart proceeds in four steps using a novel graph algorithm. Given a set of genomes:

1. **Find "HGT-genes"**: hogart detects genes in your dataset that have undergone horizontal transfer. This is done by selecting highly similar genes across highly dissimilar genomes (an "implicit phylogenetic" approach). 
2. **Create a cross mobilome graph (CMG)**: the CMG is a pangenome graph where each gene is a node and edges occur between adjacent genes, but restricted to only HGT-genes. 
3. **Pathfinding in the CMG**: paths in the CMG represent sequences of adjacent genes that have undergone HGT. We find paths with high score using a combinatorial search algorithm. 
4. **Extracting paths into modules**: the paths are processed, cleaned, and checked for additional variation. The final paths are output as a sequence of genes along with their positions in original genomes. 

