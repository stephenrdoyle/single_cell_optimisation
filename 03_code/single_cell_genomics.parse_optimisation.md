# Parse single cell sequencing analysis

# https://support.parsebiosciences.com/hc/en-us/articles/17166220335636


#working dir: /nfs/users/nfs_s/sd21/lustre_link/parse_single_cell 

# setting up the analysis tools
```bash
conda create -n spipe python=3.10

conda activate spipe

```



mkdir analysis expdata genomes


```bash
split-pipe \
--mode mkref \
--genome_name hg38 \
--fasta /newvolume/genomes/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz \
--genes /newvolume/genomes/Homo_sapiens.GRCh38.109.gtf.gz \
--output_dir /newvolume/genomes/hg38



```



```bash
# Example with explicit sample definitions
split-pipe \
    --mode all \
    --chemistry v2 \     
    --genome_dir /newvolume/genomes/hg38_mm10/ \
    --fq1 /newvolume/expdata/202306_ex1/s1_S1_R1_001.fastq.gz \
    --fq2 /newvolume/expdata/202306_ex1/s1_S1_R2_001.fastq.gz \
    --output_dir /newvolume/analysis/202306_ex1_an1 \
    --sample xcond_1 A1-A6 \
    --sample xcond_2 C1-C6 \
    --sample xcond_3 D1-D6


```