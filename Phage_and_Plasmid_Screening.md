# [Phage](https://github.com/Joshua-Macleod/FME_analysis/blob/main/Phage_and_Plasmid_Screening.md#:~:text=and%20screening%20of-,phages,-Detection%20and%20screening) and [Plasmid](https://github.com/Joshua-Macleod/FME_analysis/blob/main/Phage_and_Plasmid_Screening.md#:~:text=and%20screening%20of-,plasmids,-FME_analysis/Phage_and_Plasmid_Screening.md) screening

The below outlines scripts used to identify, characterise, extract and screen regions associated with horizontal gene transfer. The objective here was to identify movement of pertinent genetic material across sequence types or species within facilities. 

## Detection and screening of phages

Characterisation of bacteriophage sequences within assembled genomes was performed using VirSorter2, following a [protocol outlined by the Sullivan lab](https://www.protocols.io/view/viral-sequence-identification-sop-with-virsorter2-5qpvoyqebg4o/v2) with minor modifications (namely the omission of the DRAMv).

Virsorter2 - first round:
```bash
for file in FME/*.fasta; do
    filename=$(basename "$file" .fasta)
    mkdir -p "virsorter2_out/${filename}_vs2"
    virsorter run --prep-for-dramv -w "virsorter2_out_0.5/${filename}_vs2" \
    -i "$file" -j 12 --min-score 0.5 all
done
```
Run checkV on virsorter2 output:
```bash
for file in virsorter2_out_0.5/*/*.fa; do
    # Extract the directory name (e.g., A006_vs2)
    dirname=$(basename "$(dirname "$file")")
    # Remove the _vs2 suffix 
    strain_name=${dirname%_vs2}
    # Create the output directory in checkv_out using the strain name
    mkdir -p "checkv_out/${strain_name}"
    # Run CheckV using the strain name as the output directory
    checkv end_to_end "$file" "checkv_out/${strain_name}" -t 6
done

# Loop through each strain directory and combine files if both exist
for strain_dir in */; do
    if [[ -f "${strain_dir}/proviruses.fna" && -f "${strain_dir}/viruses.fna" ]]; then
        cat "${strain_dir}/proviruses.fna" "${strain_dir}/viruses.fna" > "${strain_dir}/combined.fna"
    fi
done
```
Then run virsorter2 again:
```bash
for file in checkv_out/*/combined.fna; do
    dirname=$(basename "$(dirname "$file")")
    mkdir -p "virsorter2_post_checkv_out/${dirname}_vs2cv"
    virsorter run --seqname-suffix-off --viral-gene-enrich-off --prep-for-dramv -w "virsorter2_post_checkv_out/${dirname}_vs2cv" \
    -i "$file" -j 20 --min-score 0.5 all
done
```

Next, checkV and Virsorter 2 outputs were amalgamated into a spreadsheet:

```bash
mkdir -p merged_vs2_checkv

for strain in virsorter2_out_0.5/*; do
    strain_name=$(basename "$strain")  # Extract strain name from the dir

    # Define paths to virsorter and checkv files
    vs2_file="virsorter2_out_0.5/$strain_name/final-viral-score.tsv"
    checkv_file="checkv_out/$strain_name/contamination.tsv"
    
    # Define the output file path for the merged result
    merged_output="merged_vs2_checkv/merged_output_${strain_name}.tsv"

        # Ensure both files are sorted by contig_id column
        sorted_vs2=$(mktemp)
        sorted_checkv=$(mktemp)

        sort -V -k1,1 "$vs2_file" > "$sorted_vs2"
        sort -V -k1,1 "$checkv_file" > "$sorted_checkv"

        # Merge the sorted files using contig_id as the join key
        join -t $'\t' -1 1 -2 1 -a 1 -a 2 -e "NA" "$sorted_vs2" "$sorted_checkv" > "$merged_output"

        # Clean up temporary sorted files
        rm "$sorted_vs2" "$sorted_checkv"
done
```
Then, filtering was performed to remove non-viral sequences:

```bash
mkdir -p filtered_keep_virsorter

for merged_file in merged_vs2_checkv/merged_output_*.tsv; do
    # Extract strain name by removing the 'merged_output_' prefix and '.tsv' suffix
    strain_name=$(basename "$merged_file" .tsv | sed 's/^merged_output_//')
    
    # Define the output file for the filtered contigs (Keep1 and Keep2)
    output_file="filtered_keep_virsorter/filtered_keep_${strain_name}.tsv"
    
    # Filter Keep1 and Keep2 contigs and store in the output file
    awk -F'\t' '
    NR == 1 { print $0 > "'"$output_file"'" }  # Print the header
    
    # Keep1: viral_gene > 0
    $8 > 0 { print $0 >> "'"$output_file"'" }

    # Keep2: viral_gene = 0 AND (host_gene = 0 OR score >= 0.95 OR hallmark > 2)
    $8 == 0 && ($12 == 0 || $2 >= 0.95 || $9 > 2) { print $0 >> "'"$output_file"'" }
    ' "$merged_file"
done
```
The sequences were extracted and outputed to fasta files:
```bash
# Directory for final filtered FASTA outputs
mkdir -p filtered_final_fasta

# Loop through each strain
for strain in virsorter2_post_checkv_out/*; do
    strain_name=$(basename "$strain")

    filtered_file="filtered_keep_virsorter/filtered_keep_${strain_name}.tsv"
    fasta_file="virsorter2_post_checkv_out/$strain_name/final-viral-combined.fa"
    filtered_fasta_output="filtered_final_fasta/final_viral_combined_filtered_${strain_name}.fa"

    awk '{print $1}' "$filtered_file" | sed 's/||_1/||/' > keep_ids.txt

    awk '
    BEGIN {RS=">"; FS="\n"}
    NR == FNR {keep[$1]; next}
    $1 in keep {print ">" $0}
    ' keep_ids.txt "$fasta_file" > "$filtered_fasta_output"

    rm keep_ids.txt
done
```
Each individual detected phage was separated into individual FASTA files:

```bash
# Set the base input directory
input_dir=~/shared-team/nxf_work/st/FME/VIRAL_DETECTION/virsorter2_post_checkv_out/

# Set the base output directory for the split FASTA files
output_dir=~/shared-team/nxf_work/st/FME/VIRAL_DETECTION/virsorter2_split_phages

# Create the output directory
mkdir -p "$output_dir"

# Loop through all directories in the input directory
for dir in "$input_dir"/*/; do
    strain_name=$(basename "$dir")
    clean_strain_name=$(echo "$strain_name" | sed 's/_vs2cv//')
    fasta_file="${dir}/final-viral-combined.fa"
    strain_output_dir="${output_dir}/${clean_strain_name}"
    mkdir -p "$strain_output_dir"

    awk -v strain="$clean_strain_name" -v outdir="$strain_output_dir" '
        BEGIN { RS=">"; ORS=""; }
        {
            if (NF > 1) {
                header = $1
                sequence = $2
                split(header, arr, "\\|\\|");
                contig_name = arr[1];
                contig_type = arr[2];
                gsub(/[^a-zA-Z0-9_]/, "_", contig_name);
                file = outdir "/" strain "_" contig_name "_" contig_type ".fasta";
                print ">" header > file;
                print "\n" sequence > file;
            }
        }' "$fasta_file"
done
```
These were assessed using the full Kraken2 viral database to verify viral origin and taxonomically classify.

## Detection and screening of plasmids
