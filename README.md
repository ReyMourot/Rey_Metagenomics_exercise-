# Rey_Metagenomics_exercise-
# No demultiplexing, go directly into quality filtering. 
# Go search the fasta file into the computer (dezip the file manually before):
cd 00_RAW
# (all files are together)
cp /mnt/c/Users/Audrey/Downloads/filename filename.fastq
# Creation of a tab-delimited file named "samples.txt" (for each sample, change the name of the file ex: sample1.txt, sample2.txt, sample3.txt ... 01_QC, 01_QC2 etc etc)
# Quality filtering:
mkdir 01_QC
iu-gen-configs samples.txt -o 01_QC
iu-filter-quality-minoche 01_QC/Sample_01.ini

# Co-assembly :
R1s=`ls 01_QC/*QUALITY_PASSED_R1* | python -c 'import sys; print (",".join([x.strip() for x in sys.stdin.readlines()]))'`
R2s=`ls 01_QC/*QUALITY_PASSED_R2* | python -c 'import sys; print (",".join([x.strip() for x in sys.stdin.readlines()]))'`
megahit -1 $R1s -2 $R2s --min-contig-len 1000 -m 0.85 -o 02_ASSEMBLY/ -t 40
mkdir 03_CONTIGS
anvi-script-reformat-fasta 02_ASSEMBLY/final.contigs.fa -o 03_CONTIGS/contigs.fa --min-len 2500 --simplify-names --report name_conversions.txt

# Mapping:
mkdir 04_MAPPING
bowtie2-build 03_CONTIGS/contigs.fa 04_MAPPING/contigs
bowtie2 --threads 8 -x 04_MAPPING/contigs -1 01_QC/Sample_01-QUALITY_PASSED_R1.fastq -2 01_QC/Sample_01-QUALITY_PASSED_R2.fastq -S 04_MAPPING/Sample_01.sam
samtools view -F 4 -bS 04_MAPPING/Sample_01.sam > 04_MAPPING/Sample_01-RAW.bam
anvi-init-bam 04_MAPPING/Sample_01-RAW.bam -o 04_MAPPING/Sample_01.bam
rm 04_MAPPING/Sample_01.sam 04_MAPPING/Sample_01-RAW.bam

# Creating an anvio contigs database 
anvi-gen-contigs-database -f 03_CONTIGS/contigs.fa -o contigs.db -n 'An example contigs datbase'
anvi-run-hmms -c contigs.db
anvi-display-contigs-stats contigs.db

# anvi profile 
anvi-profile -i 04_MAPPING/Sample_01.bam -c contigs.db
