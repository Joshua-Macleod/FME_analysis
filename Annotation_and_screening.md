# Annotation and screening

Assembled fasta files underwent protein annotation and screening through three (or four, actually) major packages. These are [Prokka](https://github.com/tseemann/prokka), [Bakta](https://github.com/oschwengers/bakta), [ABRicate](https://github.com/tseemann/abricate) and [chewBBACA](https://github.com/B-UMMI/chewBBACA). The first two were employed for annotation ahead of manual screening and the latter two were used for mass screening and detection of genes of interest (GOI).

## Protein Annotation

Prokka and Bakta were both used throughout. Though prokka is no longer undergoing maintenance, the database(s) provided are significant in breadth and perform well alongside the newer Bakta. Bakta still undergoes updates as of writing. Installation via conda worked well with both, and database installation was painless as well.

#### Prokka  
````bash
conda activate prokka
for file in dir/*.fasta; do
    filename=$(basename "$file" .fasta)
    prokka --genus Listeria --outdir prokka_output --force --prefix "$filename" "$file"
done
conda deactivate
````
#### Bakta
````bash
conda activate bakta
for file in dir/*.fasta; do
    filename=$(basename "$file" .fasta)
    bakta --db db --genus Listeria --output bakta_output --force --keep-contig-headers --prefix "$filename" "$file"
done
````

## Mass screening and Detection of GOI

 ABRicate was for mass screening and detection of GOI. On installation, ABRicate has some databases preinstalled which are easy enough to use. Databases can also be curated for specific screenings, as performed herein. 

For ABRicate, additional databases were curated by either (a) downloading allelic sequences from [BIGSdb-Lm](https://bigsdb.pasteur.fr/listeria/) or (b) manually curated from known sequences from databases such as NCBI. Databases were created by concatanating all allelic sequences into one fasta file, changing the name of the file to `sequences`, and then invoking ABRicate:
````bash
mkdir new_db
cd new_db
mv dir_with_genes/gene*.fa .
cat gene*.fa genes.fa
mv cat.fa sequences
abricate --setupdb
````
Then, moving to the directory with the assembled genomes awaiting screening, ABRicate was used as follows.
````bash
for file in *.fa; do
abricate --db new_db --minid 40 --mincov 40 "$file" > "$file.tsv"
done
````

## Gene cluster comparison visualisation

Pathogenicity islands were further investigated using [clinker](https://github.com/gamcil/clinker). Installed using conda, the following steps were undertaken to use .gff3 and .fa files to generate comparative alignment figures. 

First, genes of interest were extracted from .gff3 files manually. This included contig headers as seen below:

````
##gff-version 3
##feature-ontology https://github.com/The-Sequence-Ontology/SO-Ontologies/blob/v3.1/so.obo
# organism Listeria
# Annotated with Bakta
# Software: v1.9.3
# Database: v5.1, full
# DOI: 10.1099/mgen.0.000685
# URL: github.com/oschwengers/bakta
##sequence-region contig00004 1 267839
contig00004	Prodigal	CDS	26902	28311	.	-	0	ID=JKJLDO_08750;Name=Putative anchored-membrane protein;locus_tag=JKJLDO_08750;product=Putative anchored-membrane protein;Dbxref=SO:0001217,UniRef:UniRef50_E3ZLM3,UniRef:UniRef90_E3ZLM3
contig00004	Prodigal	CDS	28451	29338	.	-	0	ID=JKJLDO_08755;Name=Phospholipase C;locus_tag=JKJLDO_08755;product=Phospholipase 
````

This was combined by extracting the contig of interest from the full fasta sequence, and creating a new contig specific .fa file:

````
>contig00004 gumpf
ACTGCAGAGCTGCGGCATAGCCTTCGCAGTATCGGCTTGATACGCGCTTGTGTGTGCTCGAC
````

Clinker was then invoked using the following command:
````
clinker file1.gff3 file2.gff3 -p file_out.html
````

This generates a nice html file that can be tinkered with manually.
