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
gubbins.py *.fa
```

RAxML-NG was used to generate the phylogeny
```
```

### SNP-distance analysis

SNP distances were determined for isolates of a shared ST that were spatially and/or temporally related. These were usually clustered within a room or within a zone of a facility. Analysis was performed using Snippy, and visualised using a local instance of GrapeTree. 
