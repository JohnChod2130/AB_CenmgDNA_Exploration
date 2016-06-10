Three main types of PKS seen here http://www.scripps.edu/shen/pdfs/44.pdf. Each has a particular domain that we can use to generate a functional gene of interest for Xander. 

Let's first focus on the KS domain in Type I PKS. 
Went here http://www.clustermine360.ca/Browse.aspx
And starting downloading the sequence files for antibiotics in TIPKS-Modular. These we peptide sequences!

At the moment, we can find all of these files here: /Users/JohnChod2130/Downloads/ClusterMiner360_PKSTI_Modular/PKS_TypeI_modular_KSdomain

Specifically, I wanted to grab KS domains from each organism so I created a shell script. 
```
for i in */ ; do 
	cd "$i"
	cp *KS_* /Users/JohnChod2130/Downloads/ClusterMiner360_PKSTI_Modular/K-SDomain	
	cd ..;
 done
```

This script looked for KS domains in the directory for each organism and pooled them into a new directory -> K-SDomain. 

For Xander, we had to generate the seed sequence and the hmm sequence.. 

Fasta files were concatenated 
cat *.fasta > /outoutdirectory/KScombined.fasta
***Now I notice redundancy in some of these domains. I don't think they hurt the hmm build process but it probably can be removed at a later time. 

Considering redundancy in sequences, I ran into a problem trying to use Kalign http://www.ebi.ac.uk/Tools/msa/kalign/, so to circumvent this, I used bbmaps to change the heads of all sequences to a sequentenial order (e.g. Domain 1,2,3, etc.). 

bbrename.sh in=KScombined.fasta out=KS_Domains_bbmapRenamed.fasta prefix=domain 

Then, the KS_Domains_bbmapRenamed.fasta file was used to run Kalign. Output- KSalignment_Kalign.fasta *** This is seeds for Xander***

hmmer3.0 was downloaded (instead of the most updated versision because I don't know if that is compatible with Xander yet) and used to build the hmm. 

hmmbuild --informat afa  KS_Domains.hmm KSalignment_Kalign.fasta  ***this is the .hmm file for Xander***

KScombined.fasta or KS_Domains_bbmapRenamed.fasta is the framebot.fa file for Xander (I think it was the first). 

We also need a nucl.fa file for Xander. This was performed by taking our KS_Domains_bbmapRenamed.fasta and using Kalign to go from protein to nucleotide sequence.  
EMBOSS Backtranseq was used to back translate protein to nucleotide sequences 

http://www.ebi.ac.uk/Tools/st/emboss_backtranseq/

Considering most of these came from Strepomyces, Streptomyces coelicolor A32 was used for the condon usage table...Possible GC bias???

The file had too many sequences so it was split and then concatenated back together afterwards: 
Inputs: KS_Domains_bbmapRenamed_Seq1-200.fasta and KS_Domains_bbmapRenamed_Seq201-616.fasta
Output: KS_Domains_Nucleotides_Seq1-200.fasta and KS_Domains_Nucleotides_Seq201-616.fasta
Final concatenated file: KSnucleotides.fasta -> nucl.fa for Xander. 

We now have all four inputs for Xander for prepare gene ref. Consider putting more nucl and framebot sequences in next time. This is supposed to cover a larger breadth of diversity than the seed sequences used to build the hmm. 

Finishing writign Xander input here. 

After Xander was run, the files we care about are, test_k45_final_prot.fasta and test_k45_final_nucl.fasta. 

BWA was used to map these hits to mgDNA scaffolds

bwa index ref.fa (your scaffold reference sequence)

outputs: Cen01.scaffolds.fasta.amb Cen01.scaffolds.fasta.ann Cen01.scaffolds.fasta.bwt  prepare_gene_ref.sh Cen01.scaffolds.fasta.pac Cen01.scaffolds.fasta.sa

bwa mem ref.fa reads.fq > aln-se.sam   (reads.fq = test_k45_final_nucl.fasta).

Output: Cen01_bwamem.sam 

Need to use mem from BWA because this can handle our query sequences (Align 70bp-1Mbp query sequences with the BWA-MEM algorithm. Briefly, the algorithm works by seeding alignments with maximal exact matches (MEMs) and then extending seeds with the affine-gap Smith-Waterman algorithm (SW).). Our nucleotide sequences are about 1Kbp. 

Use SamTools to Convert BAM to SAM.
module load GNU/4.8.3
module load SAMTools/1.3
```
samtools view -b -S Cen01_bwamem.sam > Cen01_bwamem.bam
```

According to IGV: IGV requires that BAM files be sorted and indexed by coordinate. Indexing produces a secondary file with  a ".bai" extension. The resulting file is associated with the alignment track by file naming convention, or loaded independently with the index query parameter.

So to sort the bam file
```
samtools sort -O bam -T /mnt/research/ShadeLab/Chodkowski/RDPTools/Xander_assembler/Xander_on_PKS_KSdomains_rawgDNA/BWA_Mapping Cen01_bwamem.bam -o Cen01_bwamem.sorted.bam
```

Consider converting and sorting in one go 
samtools view -bS file.sam | samtools sort - file_sorted

Index sorted file to creat associated bai file 
```
samtools index Cen01_bwamem.sorted.bam
``` 
The sam output will be used to visualize alignment to scaffolds using igv. 

***Make sure when you sign into the HPCC that you use -X to allow for Xquartz to open the IGV gui. 
e.g. ssh -X chodkows@hpcc.msu.edu

module load IGV/2.3.57
igv

GUI should open. 

The current problem: MY sorted bam file looses the header and hence the reference name changes so it can't locate it on IGV. The header describes the scaffold in the .sam file but loses it in the .bam. Figure this out! It's maintained from samtools view but it loses the correct references from sort. 
THis may not be the case. Look at head of the bam sort file and it seems fine. 

I tried to do this 
```
samtools view -H Cen01_bwamem.bam > header.sam
```
Then created the sort file (see code above) 

Then 
```
samtools reheader header.sam Cen01_bwamem.sorted.bam > Cen01_bwamem.sorted1.bam
``` 
Deleted old sort and renamed new sort back to old sort name. 
Then finished with indexing and tried to upload to igv again. 

This seemed to work!
