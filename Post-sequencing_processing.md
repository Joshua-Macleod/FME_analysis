# Post-sequencing QC and Preprocessing

Ahead of any downstream analysis, fastq sequences underwent QC and preprocessing. Packages were installed using mamba where possible, conda where not. 

### Fastp
Fastp was used for a catch-all all-in-one formating of *.fastq files. 

```bash
mkdir outdir
for file in *_R1.fastq; do
  filename=$(basename "$file" _R1.fastq) # This removed the suffix, leaving only the filename
  fastp -i "${filename}_R1.fastq" -I "${filename}_R2.fastq" \
        -o "outdir/${filename}_R1.fastq" -O "outdir/${filename}_R2.fastq \
        -q 20"
done
````
### First Shovill assembly
Resulting fastq files were _de novo_ assembled using shovill ahead of screening for contaminants using kraken2. Minimal quality thresholds were set here, as the goal is to look for contaminants and not create nice clean assemblies! 

```bash
mkdir shovill_out
for file in *_R1.fastq; do
  filename=$(basename "$file" _R1.fastq) # Same as before, removing the suffix
  shovill --outdir "shovill_out/$filename" --R1 "${filename}_R1.fastq" --R2 "${filename}_R2.fastq" --force
done
```
### Kraken2
Kraken2 databases were kindly pre-installed on CLIMBS'S jupyter notebook. In an instance whereby it is not installed, the following command can be used to install particular databases (Curiously, I was about halfway through the below when I realised they came pre-installed...)

```bash
kraken2-build --download-taxonomy --db bacteria
kraken2-build --download-library bacteria --db bacteria
kraken2-build --build --db bacteria
```
Kraken2 was then invoked on the newly assembled fasta files.

```bash
dbpath="kraken2/standard_16gb"   # Kraken2 database
indir="shovill_out" 
outdir="kraken2_out"

mkdir -p $outdir
for fasta_file in $indir/*.fasta; do
    base_name=$(basename $fasta_file .fna)
    kraken2 --db $dbpath \
            --output $outdir/${base_name}_kraken2_output.txt \
            --report $outdir/${base_name}_kraken2_report.txt \
            --classified-out $outdir/${base_name}_classified.fasta \
            --unclassified-out $outdir/${base_name}_unclassified.fasta \
            *_combined.fasta*
done
```
From this, a summarised single tsv file was curated to allow for easier filtering downstream. First, it echos to the summary file a header for each column. Then, it iteratively extracts from the input report files and adds it to the new summary file.

```bash
outdir="kraken2_out"
summary_file="$outdir/kraken2_summary.tsv"

echo -e "Genome\tTaxonomy\tPercentage\tReads_Count\tTaxonomy_ID" > $summary_file

for file in $outdir/*_kraken2_report.txt; do 
    filename=$(basename $file _kraken2_report.txt)
    awk -v genome=$filename 'NR>1 {print genome "\t" $5 "\t" $1 "\t" $2 "\t" $6}' $report >> $summary_file
done
```
The resulting summary file was then used to filter out unwanted taxids from the fasta files using **seqtk** 

### QUAST

QUAST can also be used for post-assembly quality evalusation, providing an easy tool for determining alignment relative to a reference sequence. It also highlights misassemblies and genome fraction (%). 

```bash
for genome in *.fa; do
  genome_name=$(basename $genome .fa)
  quast -o "quast_results/$genome_name" "$genome" -r reference.fa
done
```

