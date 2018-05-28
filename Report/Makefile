# create a symbolic link which can point to the file in server. 
.PHONY: fqs
fqs: read1.fq read2.fq  

read1.fq:/storage/datasets/Coptotermes_formosanus/fastq/pooled_trim.headcrop16.pair1.fastq 
	ln -s $< $@

read2.fq:/storage/datasets/Coptotermes_formosanus/fastq/pooled_trim.pair2.fastq 
	ln -s $< $@

ref.fa:/storage/datasets/Coptotermes_formosanus/references/M_girerdii_genome_nt.fas
	ln -s $< $@


# ngm-utils to interleave to match the mates 
# inter.fq: read1.fq read2.fq
#	ngm-utils interleave -1 $< -2 $(word 2,$^) -o $@

# map pair-end reads with the reference genome using ngm
# building senstivity gradient in NGM
.PHONY: sams
sams: SAM/outcome_1.sam SAM/outcome_2.sam SAM/outcome_3.sam SAM/outcome_4.sam SAM/outcome_5.sam

SAM/outcome_1.sam: read1.fq read2.fq ref.fa
	ngm -1 $< -2 $(word 2,$^) -r $(word 3,$^) -o $@ --hard-clip --max-read-length 200 --skip-mate-check -t 120 -s 0.1 

SAM/outcome_2.sam: read1.fq read2.fq ref.fa
	ngm -1 $< -2 $(word 2,$^) -r $(word 3,$^) -o $@ --hard-clip --max-read-length 200 --skip-mate-check -t 120 -s 0.2

SAM/outcome_3.sam: read1.fq read2.fq ref.fa
	ngm -1 $< -2 $(word 2,$^) -r $(word 3,$^) -o $@ --hard-clip --max-read-length 200 --skip-mate-check -t 120 -s 0.3

SAM/outcome_4.sam: read1.fq read2.fq ref.fa
	ngm -1 $< -2 $(word 2,$^) -r $(word 3,$^) -o $@ --hard-clip --max-read-length 200 --skip-mate-check -t 120 -s 0.4

SAM/outcome_5.sam: read1.fq read2.fq ref.fa
	ngm -1 $< -2 $(word 2,$^) -r $(word 3,$^) -o $@ --hard-clip --max-read-length 200 --skip-mate-check -t 120 -s 0.5

# OR fancy way of doing it--for loop
target$$nums.sam:read1.fq read2.fq ref.fa
	for nums in 0.1 0.2 0.3 0.4 0.5 ; do \
	ngm -1 $< -2 $(word 2,$^) -r $(word 3,$^) $@ --hard-clip --max-read-length 200 --skip-mate-check -t 90 -s $$nums; \
	done


# samtools to extract mapped sequences
outcome.bam: SAM/outcome_3.sam
	samtools view -bS -h $< > $@

#################################################################################strategy 1 failed.

# velvet de novo assembly with the reference using mapped fastq from samtools.
mapped.sam: outcome.bam
	samtools view -F 4 -h $< > $@
sort_mapped.sam: mapped.sam
	sort $< > $@

# recompile the velvet, change the LONGSEQUENCES option.(./velveth) : *fix the bug*. 
Roadmaps Sequences: ref_trimID.fa sort_mapped.sam
	velveth autoNew_21 21 -reference $< -shortPaired -sam $(word 2,$^) 
contigs.fa stats.txt Lastgraph: auto_21
	velvetg $< -exp_cov auto

# blat mapping contigs.fa with ref genome.
blat.psl: ref.fa autoNew_21/contigs.fa
	blat $< $(word 2,$^) -minIdentity=0 -minScore=0 $@ 

# Testing the query seq mapping quality using mummer
mummer.mums: ref.fa autoNew_21/contigs.fa
	mummer -mum -c -b  $< $(word 2,$^) > $@
mummer1.ps: mummer.mums 
	mummerplot -x [0:618983] -y [0:200] -c -r CP007711.1 -postscript -p mummer1 $<
mummer2.ps: mummer.mums 
	mummerplot -x [0:629049] -y [0:200] -c -r CP020122.1 -postscript -p mummer2 $<


############################################################################ strategy 2 succeed.

# MetaSPAdes de novo assembly, We tried -K 21 55 77 and decide to use K = 21. 
spades_output1: read1.fq read2.fq
	spades.py --meta -k 21 --pe1-1 $< --pe1-2 $(word 2,$^) -t 90 -o $@

# Building symbiont assembly database via blast
# Build a db folder then run the makefile within the folder
M_girerdii.nhr: CP020122.1.fasta
	makeblastdb -in $< -title CP020122.1 -dbtype nucl -out M_girerdii -parse_seqids

blastn_CP020.txt: spades_output1/K21/final_contigs.fasta
	blastn -query $< -db CP020122.1/M_girerdii -out $@ -evalue 1e-30 -outfmt 6 -task blastn

blastn_CP007.txt: spades_output1/K21/final_contigs.fasta
	blastn -query $< -db CP007711.1/M_girerdii -out $@ -evalue 1e-30 -outfmt 6 -task blastn

# extract all the hits.
contig.nhr: ../spades_output1/K21/final_contigs.fasta
	makeblastdb -in $< -title contig_db -dbtype nucl -out contig -parse_seqids

hits2.fasta: hits2.txt
	blastdbcmd -db contig_db/contig -dbtype nucl -entry_batch $< -outfmt %f -out $@ 

hits1.fasta: hits1.txt
	blastdbcmd -db contig_db/contig -dbtype nucl -entry_batch $< -outfmt %f -out $@ 



# R script to create a graphic summary of hits.fasta from BLAST report. 

# you can try blast-imager.pl to create a graphical summary of a BLAST report, however only for partial fragment of whole chromosome.(latested version not support GIF)Not helpful. 




## clean	:Remove auto-generated files
.PHONY:clean
clean:
	rm -f mummer*.gp
	rm -f mummer*.fplot
	rm -f mummer*.rplot	

## help
.PHONY:help
help:
	@echo "inter.fq	rerdii.nhri:Gererate interleaved fastq file"
	@echo "outcome.sam	:Generate SAM files"
	@echo "mapped.sam	:Generate mapped SAM files"
	@echo "sort_mapped.sam	:Generate sorted and mapped SAM files"
	@echo "contigs.fa	:Generate velvet de novo assembly contigs"
	@echo "mummer1.ps	:Generate mummer plot"
	@echo "spades_output1	:Generate meteSPAdes de novo assembly contigs"
	@echo "M_girerdii.nhr	:Generate blast database"
	@echo "blastn_CP020.txt		:Generate blast outcome of CP_020 chromosome"
	@echo "hits2.fasta	:Generate fasta file of hits of CP_020 chromosome"
	@echo "clean		:Remove auto-generated files"
	
