# Listerial Food Manufacturing Environment Analyses
<div align="center">
  <img src="./the_listerial_lad_himself.png" alt="Among the best illustrations of _Listeria monocytogenes_ you have, perhaps, ever seen" width="500">
</div>

**Uncovering the #WelshFoodListerialLandscape**

This is a collection of custom scripts and package invocations used throughout the **Listerial FME project**. This will be gradually compiled after private details, such as file paths and filenames, are removed.

The project, along with neighbouring projects that will likely find their way to either this or another repository, was principally undertaken on [CLIMB](https://www.climb.ac.uk/) and their iteration of [Jupyter's Notebooks](https://jupyter.org/). Both have been pivotal in enabling us here at Cardiff Metropolitan University (CMU) to access command line tools and python. 

The objective of this as a repository is two-fold. First, I'd like to archive the project, tools used and enhance the reproducibility of this work. Second, I'd like to provide this to students, ECRs and anyone a little unsettled at the scary prospect of computational biology and bioinformatics. It has been fantastic fun, and this may help others realise that if _I_ can do this, they can too. I'll have invariably made mistakes and missteps, but that's all part of the process. 

# Project Goals
The work undertaken and outlined herein sought to explore the:
1. **Prevalence and distribution of _Listeria_ spp.** across ready-to-eat food manufacturing environments in Wales using Multi-Locus Sequence Typing (MLST) and phylogenetic analysis.
2. **Distribution of _Listeria_ spp. within facilities** using SNP-based distance analysis, aiming to validate the suitability of this method for determining sources of contamination, persistence and pertinent vehicles.
3. **Distribution of antimicrobial resistance, stress tolerance and virulence genes** within the genome and their association with phages and plasmids.

# Scripts
1. [Preprocessing, QC and filtering](Post-sequencing_processing.md)
2. [Annotation and screening](Annotation_and_screening.md)
3. [MLST](MLST.md)
4. [Phylogenetic analysis](https://www.youtube.com/watch?v=dQw4w9WgXcQ)
5. [SNP-based distance analysis](SNP-distance_analysis.md)
6. [Plasmid, phage, and IS detection and mass screening](Phage_and_Plasmid_Screening.md)

# Packages Used
The following packages and tools (and their respective dependencies) were used in this project. Many thanks to authors and contributors - your work is greatly appreciated and a superb contribution to science.

[ABRicate](https://github.com/tseemann/abricate)  
[Bakta](https://github.com/oschwengers/bakta)  
[BIGSdb-Lm](https://bigsdb.pasteur.fr/listeria/)  
[BLAST](https://blast.ncbi.nlm.nih.gov/)  
[Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml)
[Clinker](https://github.com/phe-bioinformatics/clinker)  
[fastp](https://github.com/OpenGene/fastp)  
[GrapeTree](https://github.com/achtman-lab/GrapeTree)  
[Gubbins](https://github.com/sanger-pathogens/gubbins)  
[iToL](https://itol.embl.de/)  
[Kraken2](https://ccb.jhu.edu/software/kraken2/)  
[MLST](https://github.com/tseemann/mlst)  
[PlasmidFinder](https://cge.cbs.dtu.dk/services/PlasmidFinder/)  
[PLSDB](https://ccb-microbe.cs.uni-saarland.de/plsdb/)  
[Prokka](https://github.com/tseemann/prokka)  
[QUAST](http://quast.sourceforge.net/)  
[RAxML-NG](https://github.com/amkozlov/raxml-ng)  
[Shovill](https://github.com/tseemann/shovill)  
[Snippy](https://github.com/tseemann/snippy)

# Contacts
You've made it this far, and I'm grateful. Feel free to contact me if you have any queries, suggestions, or want to discuss _Listeria_, other foodborne pathogens or bioinformatics. 

- jmacleod@cardiffmet.ac.uk
- [Linkedin](https://www.linkedin.com/in/joshuamacleod/)
- [ZERO2FIVE Food Industry Centre](https://www.cardiffmet.ac.uk/health/zero2five/Pages/default.aspx)
- [Ozone Research Group](https://ozoneresearchgroup.co.uk/)

#### Publications Related to our Work

[Macleod, J., Beeton, M.L., Blaxland, J., (2022) An Exploration of Listeria monocytogenes, Its Influence on the UK Food Industry and Future Public Health Strategies. _Foods_, doi: 10.3390/foods11101456](https://pmc.ncbi.nlm.nih.gov/articles/PMC9141670/)
