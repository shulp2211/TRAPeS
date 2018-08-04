# TRAPeS

We present TRAPeS (TCR Reconstruction Algorithm for Paired-End Single-cell), a software for reconstruction of T cell receptors (TCR) using short, paired-end single-cell RNA-sequencing. <br />  

TRAPeS reconstruct the TCR in 3 steps: For each chain, it first identify the V and J segments by searching for paired reads with one read mapping to the V segment and its mate mapping to the J segment. Then, a set of putative CDR3-originating reads are identified as the set of unmapped reads whose mates map to the V,J and C segments. Finally, an iterative dynamic programming algorithm is used to reconstruct the CDR3 region with the putative CDR3 reads. <br />  

For more information, see our paper in *Nucleic Acids Research* [here](https://academic.oup.com/nar/article-lookup/doi/10.1093/nar/gkx615) or our bioRxiv preprint [here](http://biorxiv.org/content/early/2016/08/31/072744)

##installing TRAPeS <br />  

TRAPeS is written in Python and C++, and currently works on Linux. TRAPeS require the following python libraries: <br />  

-	biopython  <br />
-	pysam  <br />

In addition, TRAPeS requires the following software:  <br />

-	bowtie2  <br />
-	RSEM  <br />
- samtools <br />

### before running TRAPeS  <br />
TRAPeS takes as input mapped and unmapped files of reads after genomic alignment (e.g. using TopHat).<br />
TRAPeS assumes a certain folder structure: It assumes that each cell has its own folder, and all of those folders are under one path. Also, it assumes that each cell folder has identical subfolder structure.<br />

 
##Running TRAPeS <br />

To run TRAPeS, simply run:<br />
 
>python  trapes.py \[options\] <br />

To display help: <br />

>python  trapes.py -h <br />
 
###Options when running TRAPeS <br />

**Input files:** <br />
<br />

-path : The for the folder with all the single cell samples. Assumes every subfolder under this path is a folder of a single cell sequencing results. <br />
-bam : The location of the sorted mapped bam file relative to the single cell folder. For example, if under each single cell folder there is a subfolder named “TopHat\_Output” and the mapped file is under that folder and named mapped.bam, the command should read “–bam TopHat\_Output/mapped.bam”. <br />
-unmapped : the location of the bam file containing the unmapped reads. Similar to the –bam tag, it is relative to the single cell folder. <br />
<br />
**Output files:** <br />
<br />
-output : Output prefix for the files generated by TRAPeS in every single cell folder, relative to the single cell folder (e.g. TopHat_Output/TRAPeS/TCR.out). Can include a subdirectory (TRAPeS will create that subdirectory if it does not yet exists). <br />

-sumF : Prefix for the output summary files of the entire path (summary of all single cells together). <br />
<br />
**Parameters for the TCR reconstruction** <br />
<br />
-score : The threshold score for the alignment of a read to the V or J segment. Any putative CDR3-originating read that its alignment to the V or J segments passes this score will be used for reconstruction. Default is 15, but should be changed based on sequencing quality and length. In the next version the default will be computed based on the length of the input reads <br />

-overlap : Minimum number of bases that the extended V and J segments should overlap in order to stop the reconstruction. Default is 10. <br />

-iterations : Maximum number of times to extend the V and J segments. If after that number of iterations the V and J segments still do not overlap the reconstruction is determined as unsuccessful. Default is 20. <br />  

-bases : Number of bases from the V and the J segments that will be used as the initial template for the reconstruction. Default is min(length(V), length(J)). <br />

-lowQ	 : By default, the putative CDR3-originating reads are identified as unmapped reads whose mate is aligned to the V/J/C segments. However, those reads can by chance be mapped to other places in the genome (usually low quality mapping). By including the –lowQ tag the set of CDR3-origintating reads will also include the reads that map to other places in the genome. We recommend using this tag. <br />

-top: Rank all the V-J pairs based on the number of mapped reads, and reconstruct only the top X number of V-J pairs. Default: reconstruct all V-J pairs. Recommended for very deep libraries or libraries when there are many possible V-J pairing. <br />

-byExp: Must be used along with the top parameter. Rank all the V-J pairs based on the number of mapped reads, but instead of reconstructing only the top X number of V-J pairs, randomly choose 2 V-J pairs with the same rank (same number of mapped reads) and reconstruct them. Then TRAPeS will move on to the next rank until the number of V-J pairs specified with top has been reconstructed. Default: off. <br />

-oneSide: Add this parameter to also search for productive reconstructions only from the extended V segment (in case of no overlap between the extended V and extended J segments). Default: off. <br />

-readOverlap: Consider only reads with that number of bases overlapping V/J/C segments as mapped reads. Default: 1. Note: This parameter is still being tested <br />

<br />

**Paths to other software:** <br />
<br />
-bowtie2 : Path to bowtie2. If not used assumes bowtie2 is in the default PATH <br />

-rsem: Path to RSEM. If not used assumes RSEM is in the default PATH <br />

-samtools: Path to samtools. If not used assumes samtools is in the default PATH <br />
<br />
**Other parameters:** <br />
<br />
-singleCell : Add this tag if you are only running TRAPeS on a single cell (not a library of many single cells). Currently not active <br />

-genome : The genome used for genomic alignment. Currently only hg38 or mm10 are supported. For mm10 with NCBI chromosome naming use mm10_ncbi. <br />

-strand : Strand orientation of the reads, options are [minus, plus, none]. For transcripts on the positive strand, to which strand does the rightmost (in genomic coordinates) mate of the read map to. Default is minus. <br />
<br /><br />

## TRAPeS output<br />
<br />
- sumF.summary.txt : Summary of the reconstruction status in each cell (successful/unsuccessful). <br />

- sumF.TCRs.txt : A list of all reconstructions (productive, unproductive and partial) of all cells. <br />
<br />
In addition, in each single cell folder you can find the following output files: <br />
<br />
-	output.\[alpha/beta\].junctions.txt : The set of V-J pairs found (before reconstruction) <br />
-	output.reconstructed.junctions.\[alpha/beta\].fa : the set of reconstructed junction. If reconstruction was unsuccessful, the partial V and J reconstructions will be separated by N’s. <br />
-	output.\[alpha/beta\].mapped.and.unammed.fa : the set of the putative CDR3-originating reads used for the reconstruction.  <br />
-	output.\[alpha/beta\].\[R1/R2\].fa : Set of paired-end reads that are aligned to the reconstructed TCRs in order to quantify the expression of each TCR using RSEM. <br />
-	output.\[alpha/beta\].rsem.out* : The output files created by RSEM. <br />
-	output.\[alpha/beta\].full.TCRs.fa : Fasta file with the full TCR sequences. <br />
-	output.\[alpha/beta\].full.TCRs.bestIso.fa : Fasta file with the full sequences of the TCRs, after choosing only the highly expressed isoform in case of more than one isoform for the V/J/C segments. <br />
-	output.summary.txt : Summary of all the reconstructed chains in this cell. <br />

<br /><br />

For any questions or comments, please email Shaked Afik, safik@berkeley.edu.
