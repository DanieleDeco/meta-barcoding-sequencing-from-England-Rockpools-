# meta-barcoding-sequencing from England Rockpools.
Bionformatic pipeline for meta-barcoding seequencing.
# 1) Trimming and quality control
## 1.CUTADAPT
```
#######################################################################
  1. Use cutadapt to select the sequences that have the desired primers
#######################################################################


#!/bin/bash

# Forward and reverse primers (IUPAC codes supported)
############for 16S###########
PRIMERF="AGAGTTTGATCMTGGCTCAG"
PRIMERR="RGYTACCTTGTTACGACTT"
PRIMERFC="CTGAGCCAKGATCAAACTCT"
PRIMERRC="AAGTCGTAACAAGGTARCY"
############for 16S###########

############for 18S###########



############for 18S###########

# Loop over all FASTQ.gz files in the current directory
for file in *.fastq.gz; do
    if [[ -f "$file" ]]; then
        base=$(basename "$file" .fastq.gz)
        echo "Trimming: $file"

        cutadapt \
            -g "$PRIMERF...$PRIMERRC" \
	-g "$PRIMERR...$PRIMERFC" \
            -o "${base}_trimmed.fastq.gz" \
            "$file" \
	--error-rate 0.2 \
	--cores=40 \
            --discard-untrimmed
    fi
done
```

## 2.NANOFILT

```
########################################################
  2.Use nanofilt to quality trimm the selected sequences
########################################################

#!/bin/bash

for file in *.fastq.gz; do
    if [[ -f "$file" ]]; then
        base=$(basename "$file" _trimmed.fastq.gz)
        echo "Trimming: $file"
        gunzip -c "$file" | NanoFilt -q 15 -l xxx --maxlength xxxx --tailcrop 50 --headcrop 50 > "${base}_filtered_crop_q15.fastq"
    fi
done

#################################
-l 800 --maxlength 1600 for 16S
-l 800 --maxlength 1900 for 18S
#################################
```
# 2) Select representative sequences assign taxonomy and construct the OTU table
##  1.Orient the reads and remove chimeras
```
vsearch --orient filtered_q15.fa --db ref_db.fasta --fastaout oriented_q15.fa
##############################
ref_db=silva_16S.fasta for 16S
ref_db=silva_16S.fasta for 16S
##############################

```
```
vsearch --uchime_ref input.fasta \
        --nonchimeras output.good.fasta \
        --chimeras output.chimeras.fasta \
        --db ref_db.fasta
##############################
ref_db=silva_16S.fasta for 16S
ref_db=silva_16S.fasta for 16S
##############################

```
## 2.Amplicon Sorter
###########################################################################################################
Use Amplicon_sorter to cluster filtered reads into groups of closely related species and generate consensus sequences.
###########################################################################################################
```
./amplicon_sorter.py   -i all_oriented.fasta -o amplicon_clustered_oriented -np 64   -min xxx   -max xxxx -maxr=xxxxx --allreads
########################################
-min 800 --max 1600 for 16S -maxr=
-min 800 --max 1900 for 18S -maxr=
########################################
```
