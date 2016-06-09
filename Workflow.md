Three main types of PKS seen here http://www.scripps.edu/shen/pdfs/44.pdf. Each has a particular domain that we can use to generate a functional gene of interest for Xander. 

Let's first focus on the KS domain in Type I PKS. 
Went here http://www.clustermine360.ca/Browse.aspx
And starting downloading the sequence files for antibiotics in TIPKS-Modular. These we peptide sequences!

At the moment, we can find all of these files here: /Users/JohnChod2130/Downloads/ClusterMiner360_PKSTI_Modular/PKS_TypeI_modular_KSdomain

Specifically, I wanted to grab KS domains from each organism so I created a shell script. 

for i in */ ; do 
	cd "$i"
	cp *KS_* /Users/JohnChod2130/Downloads/ClusterMiner360_PKSTI_Modular/K-SDomain	
	cd ..;

 done

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

bwa mem ref.fa reads.fq > aln-se.sam   (reads.fq = test_k45_final_nucl.fasta).

Need to use mem from BWA because this can handle our query sequences (Align 70bp-1Mbp query sequences with the BWA-MEM algorithm. Briefly, the algorithm works by seeding alignments with maximal exact matches (MEMs) and then extending seeds with the affine-gap Smith-Waterman algorithm (SW).). Our nucleotide sequences are about 1Kbp. 

The sam output will be used to visualize alignment to scaffolds using igv. 

