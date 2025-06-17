# Phylogenetic Analysis

Phylogenetic analysis was performed using Snippy, Gubbins and RAxML and was visualised using iTOL. SNP-distance analysis performed on select spatially related clusters were analysed as described further below. 

### Phylogeny

Fastp processing paired-end reads were aligned to a reference sequence using snippy

```bash
for file in fastp_out/*R1.fastq; do
    filename=$(basename "$file" _R1.fastq)  # Extract the filename without dir and extension  
    snippy --outdir "snippy/snippy_outdir/${filename}" --mapqual 10 --minqual 10 --mincov 5 --ref reference/reference_seq.fa --R1 "fastp_out/${filename}_R1.fastq" --R2 "fastp_out/${filename}_R2.fastq"
done
```
A core alignment was generated from individual alignments

```bash
snippy-core --prefix snippy_core_outdir snippy_outdir/* --ref reference/reference_seq.fa
```

Gubbins was used to identify and filter predicted recombinatory events 

```bash

gubbins.py snippy_core_outdir/*.fa 
```

RAxML-NG was used to generate the phylogeny
```bash
raxml-ng --all --msa *core.fa --model GTR+G --prefix tree --bs-trees 2000 --threads 6 --tree pars{25},rand{25}

raxml-ng --support --tree tree.raxml.bestTree --bs-trees tree.raxml.bootstraps --prefix tree --threads 6

raxml-ng --bsconverge --bs-trees tree.raxml.bootstraps --prefix checkingconvergence --seed 2 --threads 6 --bs-cutoff 0.03
```
Resulting phylogenies were visualised using [iTOL](https://itol.embl.de/), with annotations added using the datasets feature.

### SNP-distance analysis

SNP distances were determined for isolates of a shared ST that were spatially and/or temporally related. These were usually clustered within a room or within a zone of a facility. Analysis was performed using Snippy, and visualised using a local instance of GrapeTree. 

As above, snippy and snippy-core were used to identify positional variation (e.g. SNPs, indels) relative to a species-specific reference strain. 

```bash
for file in SNP_dist*/*.fa; do
    # Extract the filename without directory and extension
    filename=$(basename "$file" .fa)    
    # Run snippy
    snippy --outdir "snippy/${filename}" --mapqual 10 --minqual 10 --mincov 5 --ref reference/EGD-e.fasta --ctgs "$file"
done

snippy-core --prefix snippy_core snippy/* --ref reference/EGD-e.fasta
```
Resulting core .fa alignments were uploaded to a local instance of GrapeTree as available on the [GitHub repository](https://github.com/achtman-lab/GrapeTree). Metadata was uploaded to the service enabling annotation.

