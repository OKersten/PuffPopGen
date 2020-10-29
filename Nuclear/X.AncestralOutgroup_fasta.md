# Ancestral fasta


```bash
cp RAZ_Anc.Assembly_Puffin_NUMT.realigned.bam . 
# This realigned bam was produced mapping the entirety of RAZ sequence data to the puffin reference genome
# The sample was called RAZ_Anc

# min and max depth half and double ave. depth -> 33.12
module load angsd/0.931-GCC-8.2.0-2.31.1
angsd -C 50 -doFasta 2 -doCounts 1 -i RAZ_Anc.Assembly_Puffin_NUMT.realigned.bam -out Razorbill.NU.MT -setMinDepth 15 -setMaxDepth 67 -minMapQ 30 -minQ 30 -ref Assembly_Puffin_NU.MT.fasta


#No MT, Unplaced, or Z
gunzip Razorbill.NU.MT.fa.gz
cat Razorbill.NU.MT.fa | grep ">" | grep -v "scaffold1766" | grep -v "unplaced" | grep -v "chromosome_Z" | awk -F '>' '{print $2}' > Reference.NU.NoZ.NoUnpl.Sequence.names

module purge
module load seqtk/1.3-foss-2018b
seqtk subseq Razorbill.NU.MT.fa Reference.NU.NoZ.NoUnpl.Sequence.names | seqtk seq -l 80 - > Razorbill.NU.NoUnpl.NoZ.fasta

module purge 
module load SAMtools/1.9-GCC-8.2.0-2.31.1
samtools faidx Razorbill.NU.NoUnpl.NoZ.fasta

gzip Razorbill.NU.MT.fa

```