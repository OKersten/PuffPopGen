# 1. SNP + Indel Call


Get all bam files from Paleomix run and extract mitochondrial mapping from bam files

```bash

cat Modern.samples | grep -v "RAZ" | while read i ;
do echo ${i} ;
sbatch IndexSortMT.sh ${i} ;
done ;
#30 min

```

nano IndexSortMT.sh

```bash
#!/bin/bash

module load SAMtools/1.9-GCC-8.2.0-2.31.1

samtools index ${1}.Assembly_Puffin_NUMT.realigned.bam ;
samtools view -b -h -o ${1}.Assembly_Puffin_MT.bam ${1}.Assembly_Puffin_NUMT.realigned.bam scaffold1766 ;
samtools sort ${1}.Assembly_Puffin_MT.bam > ${1}.Assembly_Puffin_MT.sorted.bam ;
samtools index ${1}.Assembly_Puffin_MT.sorted.bam ;
```

Call SNPs with GATK4 - Haplotype Caller

```bash
rm *MT.bam
rm *realigned.bam*
rm *realigned.bai
rm slurm*

mkdir GATK_Data
mkdir GATK_Log

for f in $(cat Modern.samples | grep -v "RAZ") ; 
do sbatch mtDNA_SNPs_pt1.sh ${f} ; 
done

```

nano mtDNA_SNPs_pt1.sh

```bash
#!/bin/bash


module load GATK/4.1.4.0-GCCcore-8.3.0-Java-1.8

gatk --java-options "-Xmx8g" HaplotypeCaller \
-VS STRICT \
-R Assembly_Puffin_NU.MT.fasta \
-I ${1}.Assembly_Puffin_MT.sorted.bam \
-ploidy 1 \
-ERC GVCF \
-L scaffold1766 \
-O ${1}.g.vcf.gz 2> Haplotype_caller.${1}.out

mv Haplotype_caller.${1}.out GATK_Log/ ;
mv ${1}.g.vcf.gz GATK_Data/ ;
mv ${1}.g.vcf.gz.tbi GATK_Data/ ;

```

```bash
rm slurm*
rm *sorted*
```