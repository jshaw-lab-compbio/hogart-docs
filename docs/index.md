<p align="center"><img src="assets/logo.jpg" width = "400"/></p>

**Hogart** identifies shared and syntenic gene clusters, or "modules", across diverse microbial genomes. These modules can reveal **integrated mobile genetic elements (iMGEs)**, such as prophages, integrative conjugative elements, and transposons, shared between >= 2 genomes.

<p align="center"><img src="assets/input_and_outputs.png" width = "800"/></p>


### What can hogart do?

Hogart differs from horizontal gene transfer (HGT) detection methods that use phylogenetics and genomic island prediction tools. Instead, hogart (1) uses an all-to-all comparative method and (2) detects multi-gene transfers.

1. **Detect multi-gene transfers simultaneously in tens to thousands of genomes**. Existing methods detect HGT one gene at a time. However, multiple genes are usually transferred in one event (i.e., as a module). Hogart predicts **multi-gene** transfers very efficiently. 
2. **Compares all-to-all genomes and finds multi-way (i.e., not just pairwise) transfers**. Hogart outputs (1) modules shared by >= 2 genomes and (2) the loci where the modules are integrated. 
3. **Easy to use: input genomes or annotations --> modules in one command**. Hogart is also easy to install, relying on minimal dependencies. 
4. **Detects small or large transfers.** Hogart can detect iMGEs of 2 genes (e.g., transposon-like) and even ICEs and prophages of > 100 kbp. 

For more information on [how hogart works, and its limitations, see here](explanation.md).


## How do I install and run hogart?

See the left side bar for more information. See [the quick start and installation instructions](install+quickstart.md) for a quick start. 


## Changelog

See the [CHANGELOG](https://github.com/bluenote-1577/hogart/blob/main/CHANGELOG.md) for complete details.

## Citing hogart 

Jim Shaw and Heng Li. Forthcoming. 



