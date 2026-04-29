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
PRIMERF="AGAGTTTGATCMTGGCTCAG"
PRIMERR="RGYTACCTTGTTACGACTT"
PRIMERFC="CTGAGCCAKGATCAAACTCT"
PRIMERRC="AAGTCGTAACAAGGTARCY"

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
        gunzip -c "$file" | NanoFilt -q 15 -l 800 --maxlength 1600 --tailcrop 50 --headcrop 50 > "${base}_filtered_crop_q15.fastq"
    fi
done

#################################
-l 800 --maxlength 1600 for 16S
-l 800 --maxlength 1900 for 18S
#################################
```
# 2) Select representative sequences assign taxonomy and construct the OTU table

