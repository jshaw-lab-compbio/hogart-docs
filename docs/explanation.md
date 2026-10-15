### Common use cases:

You may want to use hogart to study HGT and shared integrated mobile genetic elements (iMGEs):

- between a diverse set of species (e.g., all species in a genera or family). 
- across assembled contigs / MAGs in **a single metagenome sample**
    - Works best for long-read metagenome assemblies with high contiguity; iMGEs can be poorly assembled by short reads. 
- high-quality MAGs from **different metagenomic samples**.
    - But be very careful of short-read MAG contamination that looks like HGT!

### Limitations of hogart:

- **Single-species datasets**. Hogart excels at finding HGT when the input is a **diverse set of species**. Hogart will not work if you dataset consists of only 1 species. 
    - Prefer pangenome methods like [PanRGP](https://microscope.readthedocs.io/en/stable/content/compgenomics/panRGP.html) if you want to study 1 species. 
- **Lack of HGT in dataset**. An iMGE must be shared across different species *in your dataset*. E.g., a prophage present in only 1 genome in your data will not be detected as a module. 
    - If you just want to detect putative HGT clusters in a single genome, consider genomic island (GI) prediction tools. 
- **Diverged iMGEs**. Hogart excels at finding **recently shared iMGEs** (high sequence similarity). The initial gene clustering requires 99% identity by default (`hogart run --cluster-ani`). iMGEs without sufficiently similar shared genes can be missed; the later extraction step allows more divergent genes around those anchors.

### How does hogart work?

Hogart proceeds in four steps using a novel graph algorithm. Given a set of genomes:

1. **Find "HGT-genes"**: hogart detects genes in your dataset that have undergone horizontal transfer. This is done by selecting highly similar genes across highly dissimilar genomes (an "implicit phylogenetic" approach). 
2. **Create a mobilome synteny graph (MSG)**: the MSG is a pangenome graph where each gene cluster is a node and edges link nearby HGT-genes with compatible genomic order. Non-HGT genes can occur between linked genes.
3. **Pathfinding in the MSG**: paths in the MSG represent sequences of adjacent genes that have undergone HGT. We find paths with high score using a combinatorial search algorithm.
4. **Extracting paths into modules**: the paths are processed, cleaned, and checked for additional variation. The final paths are output as a sequence of genes along with their positions in original genomes. 

