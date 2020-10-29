# Outgroup Likelihood for Trees (No Thu)

create an outgroup for all the tree analyses (NJ tree, Treemix)


```bash

# parse MT+NUC 
#module load SAMtools/1.9-GCC-8.2.0-2.31.1

samtools index -@ 8 RAZ.Assembly_Puffin_NUMT.realigned.bam
samtools view -@ 8 -b -h -o RAZ.Assembly_Puffin_NU.bam RAZ.Assembly_Puffin_NUMT.realigned.bam Scaffolds_chromosome_10 Scaffolds_chromosome_11 Scaffolds_chromosome_12 Scaffolds_chromosome_13 Scaffolds_chromosome_14 Scaffolds_chromosome_15 Scaffolds_chromosome_16 Scaffolds_chromosome_17 Scaffolds_chromosome_18 Scaffolds_chromosome_19 Scaffolds_chromosome_1 Scaffolds_chromosome_20 Scaffolds_chromosome_21 Scaffolds_chromosome_22 Scaffolds_chromosome_23 Scaffolds_chromosome_24 Scaffolds_chromosome_25 Scaffolds_chromosome_2 Scaffolds_chromosome_3 Scaffolds_chromosome_4 Scaffolds_chromosome_5 Scaffolds_chromosome_6 Scaffolds_chromosome_7 Scaffolds_chromosome_8 Scaffolds_chromosome_9 Scaffolds_chromosome_Z Scaffolds_unplaced
samtools sort -@ 8 RAZ.Assembly_Puffin_NU.bam > RAZ.Assembly_Puffin_NU.sorted.bam
samtools index -@ 8 RAZ.Assembly_Puffin_NU.sorted.bam
rm RAZ.Assembly_Puffin_NU.bam
rm RAZ.Assembly_Puffin_NUMT.realigned.bam
rm RAZ.Assembly_Puffin_NUMT.realigned.bam.bai

#then
module purge 
module load SAMtools/1.9-GCC-8.2.0-2.31.1
gunzip Razorbill.NU.MT.fa.gz
samtools faidx Razorbill.NU.MT.fa

ls ../*.bam | grep -v "IOM001" | grep -v "THU" > bams_good

#create files for main script and to get region file
echo -e "Scaffolds_chromosome_10:\nScaffolds_chromosome_11:\nScaffolds_chromosome_12:\nScaffolds_chromosome_13:\nScaffolds_chromosome_14:\nScaffolds_chromosome_15:\nScaffolds_chromosome_16:\nScaffolds_chromosome_17:\nScaffolds_chromosome_18:\nScaffolds_chromosome_19:\nScaffolds_chromosome_1:\nScaffolds_chromosome_20:\nScaffolds_chromosome_21:\nScaffolds_chromosome_22:\nScaffolds_chromosome_23:\nScaffolds_chromosome_24:\nScaffolds_chromosome_25:\nScaffolds_chromosome_2:\nScaffolds_chromosome_3:\nScaffolds_chromosome_4:\nScaffolds_chromosome_5:\nScaffolds_chromosome_6:\nScaffolds_chromosome_7:\nScaffolds_chromosome_8:\nScaffolds_chromosome_9:" > ChromosomeList.NoUn.NoZ

zcat ModernPuffinAngsd_NoZ_NoUnpl.beagle.ldpruned.gz | tail -n +2 | awk '{print $1}' - | awk -F '_' '{print $1"_"$2"_"$3"\t"$4}' > positions

cat ChromosomeList.NoUn.NoZ | while read line ;
do line_edit=$(echo ${line} | awk -F ':' '{print $1}') ;
grep -w ${line_edit} positions > ${line_edit}.positions ;
done

#run Puffin_ANGSD_main_Outgroup.sh
cat ChromosomeList.NoUn.NoZ | grep -v "Scaffolds_chromosome_1:" | grep -v "Scaffolds_chromosome_2:" | grep -v "Scaffolds_chromosome_3:" | while read i ;
do echo ${i} ;
sbatch Puffin_ANGSD_main_Outgroup.sh ${i} ;
done

```

-> Puffin_ANGSD_main_Outgroup.sh 

```bash
#!/bin/bash

Chromosome=$(echo ${1} | awk -F ':' '{print $1}' | awk -F '_' '{print $2"_"$3}')
ChromosomeAlt=$(echo ${1} | awk -F ':' '{print $1}')

echo ${Chromosome}
echo ${ChromosomeAlt}


FILTERS="-uniqueOnly 1 -remove_bads 1 -minMapQ 30 -minQ 30 -dosnpstat 1 -C 50 -baq 2 -checkBamHeaders 1 -doHWE 1 -skipTriallelic 1"
TODO="-doMajorMinor 1 -doMaf 1 -doCounts 1 -doGeno 8 -doPost 1 -doGlf 2"

module purge
module load SAMtools/1.9-GCC-8.2.0-2.31.1
samtools faidx Razorbill.NU.MT.fa

module purge
module load angsd/0.931-GCC-8.2.0-2.31.1

Threads="16"

#index the sites file
echo ${1}

sleep 120

angsd sites index ${ChromosomeAlt}.positions

sleep 120

angsd -b bams_good_edit -GL 1 $FILTERS $TODO -P ${Threads} -ref Assembly_Puffin_NU.MT.fasta -anc Razorbill.NU.MT.fa -r ${1} -sites ${ChromosomeAlt}.positions -out ModernPuffinAngsd_Outgroup_${Chromosome}


```

SUMMARY

```bash
#No Z or unplaced
zcat ModernPuffinAngsd_Outgroup_chromosome_1.beagle.gz > ModernPuffinAngsd_Outgroup_NoZ_NoUn.beagle 
for i in 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 2 3 4 5 6 7 8 9 ;
do zcat ModernPuffinAngsd_Outgroup_chromosome_${i}.beagle.gz | tail -n +2 >> ModernPuffinAngsd_Outgroup_NoZ_NoUn.beagle ;
done ;
gzip ModernPuffinAngsd_Outgroup_NoZ_NoUn.beagle
zcat ModernPuffinAngsd_Outgroup_NoZ_NoUn.beagle.gz | tail -n +2 | wc -l

#grep Outgroup column and attach to original standalone beagle pruned
# RAZ is ind. #54  - 3 columns per ind. + 3 columns for position/allele1+2
# -> columns 163, 164, 165
echo -e "RAZ\tRAZ\tRAZ" > OutgroupLikelihoods
zcat ModernPuffinAngsd_Outgroup_NoZ_NoUn.beagle.gz | awk '{print $163"\t"$164"\t"$165}' | tail -n +2 >> OutgroupLikelihoods

zcat ModernPuffinAngsd_NoZ_NoUnpl.beagle.ldpruned.gz | paste - OutgroupLikelihoods > ModernPuffinAngsd_NoZ_NoUnpl.beagle.ldpruned.withOut
gzip ModernPuffinAngsd_NoZ_NoUnpl.beagle.ldpruned.withOut  
  
```