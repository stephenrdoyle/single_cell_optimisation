# Parse single cell sequencing analysis

- Stephen Doyle





## Samples

| Sample number | Species                | Sex/stage    | Sample_name             | Plate location |
|---------------|------------------------|--------------|-------------------------|----------------|
| 1             | Caenorhabditis elegans | mixed adults | celegans_adult_mixed    | A1,A7          |
| 2             | Trichuris muris        | adult male   | tmuris_adult_male       | A2,A8          |
| 3             | Trichuris muris        | adult female | tmuris_adult_female     | A3,A9          |
| 4             | Haemonchus contortus   | adult male   | hcontortus_adult_male   | A4,A10         |
| 5             | Haemonchus contortus   | adult female | hcontortus_adult_female | A5,A11         |
| 6             | Haemonchus contortus   | L3           | hcontortus_L3           | A6,A12         |








# Working directory and setup
```bash
cd /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell 

```

## Setting up the reference files
- Haemonchus contortus
- Trichuris muris
- Caenorhabditis elegans

### Haemonchus
```bash
# reference directory
cd /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/genomes

#------ Haemonchus
wget https://ftp.ebi.ac.uk/pub/databases/wormbase/parasite/releases/WBPS18/species/haemonchus_contortus/PRJEB506/haemonchus_contortus.PRJEB506.WBPS18.genomic.fa.gz

wget https://ftp.ebi.ac.uk/pub/databases/wormbase/parasite/releases/WBPS18/species/haemonchus_contortus/PRJEB506/haemonchus_contortus.PRJEB506.WBPS18.annotations.gff3.gz

gunzip haemonchus_contortus.PRJEB506.WBPS18.annotations.gff3.gz



# convert the gff to gtf
conda activate agat

agat_convert_sp_gff2gtf.pl --gff haemonchus_contortus.PRJEB506.WBPS18.annotations.gff3 --gtf_version 3 --output  haemonchus_contortus.PRJEB506.WBPS18.annotations.gtf

conda deactivate


# need to make a fix - split-pipe needs a "gene_biotype"
sed -i 's/; biotype/; gene_biotype/' haemonchus_contortus.PRJEB506.WBPS18.annotations.gtf


# run spipe make reference
conda activate spipe

split-pipe \
--mode mkref \
--genome_name hcontortus \
--fasta haemonchus_contortus.PRJEB506.WBPS18.genomic.fa.gz \
--genes haemonchus_contortus.PRJEB506.WBPS18.annotations.gtf \
--output_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/genomes/hcontortus

```

### Trichuris muris
```bash
wget https://ftp.ebi.ac.uk/pub/databases/wormbase/parasite/releases/WBPS18/species/trichuris_muris/PRJEB126/trichuris_muris.PRJEB126.WBPS18.genomic.fa.gz

wget https://ftp.ebi.ac.uk/pub/databases/wormbase/parasite/releases/WBPS18/species/trichuris_muris/PRJEB126/trichuris_muris.PRJEB126.WBPS18.annotations.gff3.gz

# gff to gtf
conda activate agat

agat_convert_sp_gff2gtf.pl --gff trichuris_muris.PRJEB126.WBPS18.annotations.gff3.gz --gtf_version 3 --output  trichuris_muris.PRJEB126.WBPS18.annotations.gtf

conda deactivate

# filter 
grep "WormBase" trichuris_muris.PRJEB126.WBPS18.annotations.gtf > wb.gtf
mv wb.gtf trichuris_muris.PRJEB126.WBPS18.annotations.gtf

# need to make a fix - split-pipe needs a "gene_biotype"
sed -i 's/; biotype/; gene_biotype/' trichuris_muris.PRJEB126.WBPS18.annotations.gtf


split-pipe \
--mode mkref \
--genome_name tmuris \
--fasta trichuris_muris.PRJEB126.WBPS18.genomic.fa.gz \
--genes trichuris_muris.PRJEB126.WBPS18.annotations.gtf \
--output_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/genomes/tmuris

```

## Caenorhabditis celegans

```bash

wget https://ftp.ebi.ac.uk/pub/databases/wormbase/parasite/releases/WBPS18/species/caenorhabditis_elegans/PRJNA13758/caenorhabditis_elegans.PRJNA13758.WBPS18.genomic.fa.gz

wget https://ftp.ebi.ac.uk/pub/databases/wormbase/parasite/releases/WBPS18/species/caenorhabditis_elegans/PRJNA13758/caenorhabditis_elegans.PRJNA13758.WBPS18.annotations.gff3.gz


# gff to gtf
conda activate agat

agat_convert_sp_gff2gtf.pl --gff caenorhabditis_elegans.PRJNA13758.WBPS18.annotations.gff3.gz --gtf_version 3 --output  caenorhabditis_elegans.PRJNA13758.WBPS18.annotations.gtf

conda deactivate

grep "WormBase" caenorhabditis_elegans.PRJNA13758.WBPS18.annotations.gtf > tmp
mv tmp caenorhabditis_elegans.PRJNA13758.WBPS18.annotations.gtf

# need to make a fix - split-pipe needs a "gene_biotype"
sed -i 's/; biotype/; gene_biotype/' caenorhabditis_elegans.PRJNA13758.WBPS18.annotations.gtf

split-pipe \
--mode mkref \
--genome_name celegans \
--fasta caenorhabditis_elegans.PRJNA13758.WBPS18.genomic.fa.gz \
--genes caenorhabditis_elegans.PRJNA13758.WBPS18.annotations.gtf \
--output_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/genomes/celegans

```







## Get the data

```bash
# data dir 
mkdir ~/lustre_link/parse_single_cell/expdata/skb_run1

cd ~/lustre_link/parse_single_cell/expdata/skb_run1


# download cram from iRODs
iget 48163_1#1

# convert the cram to fastq files
/nfs/users/nfs_s/sd21/bash_scripts/run_cram2fastq


# remove the cram
rm *cram

# check read quality with fastqc and multiqc

bsub.py 10 fastqc "~sd21/bash_scripts/run_fastqc"

scp sd21@farm5-head2:~/lustre_link/parse_single_cell/expdata/skb_run1/multiqc_report.html skb_run1_multiqc_report.html
```

[Multiqc report](../04_analysis/skb_run1_multiqc_report.html) 





## Run the pipeline
```bash

# celegans
echo -e "
source /nfs/users/nfs_s/sd21/lustre_link/software/anaconda3/bin/activate spipe

split-pipe \
--mode all \
--chemistry v2 \
--nthreads 20 \
--fq1 /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/expdata/skb_run1/48163_1#1_1.fastq.gz \
--fq2 /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/expdata/skb_run1/48163_1#1_2.fastq.gz \
--output_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/analysis/skb1_celegans \
--genome_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/genomes/celegans \
--sample celegans_adult_mixed A1,A7" > run_skb1_celegans.sh


bsub -q long -E 'test -e /nfs/users/nfs_s/sd21' -R "select[mem>10000] rusage[mem=10000]" -n 20 -M10000 -o parse_skb1_celegans.o -e parse_skb1_celegans.e -J parse_skb1_celegans  < run_skb1_celegans.sh


# hcontortus
echo -e "
source /nfs/users/nfs_s/sd21/lustre_link/software/anaconda3/bin/activate spipe

split-pipe \
--mode all \
--chemistry v2 \
--nthreads 20 \
--fq1 /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/expdata/skb_run1/48163_1#1_1.fastq.gz \
--fq2 /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/expdata/skb_run1/48163_1#1_2.fastq.gz \
--output_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/analysis/skb1_hcontortus \
--genome_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/genomes/hcontortus \
--sample hcontortus_adult_male A4,A10 \
--sample hcontortus_adult_female A5,A11" > run_skb1_hcontortus.sh

bsub -q long -E 'test -e /nfs/users/nfs_s/sd21' -R "select[mem>10000] rusage[mem=10000]" -n 20 -M10000 -o parse_skb1_hcontortus.o -e parse_skb1_hcontortus.e -J parse_skb1_hcontortus  < run_skb1_hcontortus.sh


# tmuris
echo -e "
source /nfs/users/nfs_s/sd21/lustre_link/software/anaconda3/bin/activate spipe

split-pipe \
--mode all \
--chemistry v2 \
--nthreads 20 \
--fq1 /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/expdata/skb_run1/48163_1#1_1.fastq.gz \
--fq2 /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/expdata/skb_run1/48163_1#1_2.fastq.gz \
--output_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/analysis/skb1_tmuris \
--genome_dir /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell/genomes/tmuris \
--sample tmuris_adult_male A2,A8 \
--sample tmuris_adult_female A3,A9" > run_skb1_tmuris.sh

bsub -q long -E 'test -e /nfs/users/nfs_s/sd21' -R "select[mem>10000] rusage[mem=10000]" -n 20 -M10000 -o parse_skb1_tmuris.o -e parse_skb1_tmuris.e -J parse_skb1_tmuris  < run_skb1_tmuris.sh



```






Parse - Run 2
| Sample number | Species                | Sex/stage    | Sample_name             | Plate location |
|---------------|------------------------|--------------|-------------------------|----------------|
| 1             | Haemonchus contortus   | adult female | hcontortus_adult_female | A1,A2          |
| 2             | Haemonchus contortus   | adult male   | hcontortus_adult_male   | A3,A4          |
| 3             | Trichuris muris        | adult female | tmuris_adult_female     | A5,A6          |
| 4             | Trichuris muris        | adult male   | tmuris_adult_male       | A7,A8,A9       |
| 5             | Caenorhabditis elegans | mixed adults | celegans_adult_mixed    | A10,A11        |
| 6             | Haemonchus contortus   | L3           | hcontortus_L3           | A12            |
