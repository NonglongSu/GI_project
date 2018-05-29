The original reference genome is in /storage/datasets/Coptotermes_formosanus/references ; 
The paired-end sequencing files are in  /storage/datasets/Coptotermes_formosanus/fastq on our lab server. 

Methodology:
1. To build a convenient working environment you can create a symbolic link to the original file address (save storage space on your server), and rename the reference as ref.fa,  sequence files as read1.fq and read2.fq. 

2. There are tons of mappers we could use for the dataset. However, before randomly choosing a mapper tool we decide to use Teaser which is a tool that analyze the performance of read mappers based on specific data set. Testing results showed that NGM mapper provides the highest percentage of mapped reads compared to BWA-MEM, BWA-SW, Bowtie 2, and BWA.

3.  In order to obtain the highest percentage of mapping reads, we use NGM ( with reference) with a range of sensitivity gradient [0.1, 0.2, 0.3, 0.4, 0.5] and produce the best outcome (1.7% mapped reads) when s = 0.3.

4. 
Strategy 1:
Only using the mapped reads from two original files to conduct de novo assembly with reference genome. 

Velvet de novo assembly creates a contigs.fa with all mapped reads. Then we use  blat to produce an blast alignment file and create a sequencing mapping quality graph via mummer.                  

Velvet de novo assembly logfile showed that the value of N50 (50% of the nucleotides will be in contigs this size or larger) is low and len_max (the biggest contig) is short. 

So we decide to try a second strategy which is expected to get a higher N50 and Len_max.

Strategy 2:
Directly create an assembly file from original fastq files and blast the assembly outcome with reference genome later. 

The difference of application of this method is to avoid using NGM mapper tool and velvet to cost us much less time. Therefore, we decide to choose MetaSPAdes de novo assembly which allows us to choose the kmer length based on our preference. I choose default kmer length = 21. 

The logfile outcome is much better compared with strategy 1 [ based on mapped reads percentage and true numbers]. 

5. Continued as strategy 2, I use the blast+ assignment tool to deal with the contigs.fa file from MetaSPAdes. You have to build a database based on separate reference genomes [ CP007711.1, CP020122.1]and obtained the blastn .txt outcome which contains all alignment information [identity alignment%, seq length, location of the mapping reads …]. 

6. Finally,  We write several R scripts to create two graphical plots which display the location of hits vs reference genome and pull out all of mapped reads and convert it into fasta file via blast+. 


