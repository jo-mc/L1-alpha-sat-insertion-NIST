# L1-alpha-sat-insertion-NIST

Task: Jakob Heinz
Hi all, I was working on a global alignment of the sequence around the breakpoints of reads supporting the same events in HG008T for my tool and noticed a variable-length polyT insertion between the chr2 to chr1-aSat hap2 translocation, which is a pretty common scar of L1 insertion and indeed most of the reads have 13bps of L1 sequence after the polyT, with 3bps of microhomology (AAG) to the chr1-aSat breakpoint, so it looks like it might be some sort of L1 mediated translocation (which I do not know much about!).  Justin suggested I ask the group (particularly @Joe Mcconnell) to see if anyone has looked into this or has any insights? I know there’s a reported L1 somatic insertion in the aSat at chr1_hap2:124339632 (about 1Mb after the translocation). I had Claude annotate the read sequences here, if anyone is curious. Thanks in advance!

<img width="2890" height="2635" alt="image" src="https://github.com/user-attachments/assets/b0b35979-4286-4d04-9a40-3b31412c92f2" />


Tumor reference :

for anyone who is doing direct comparisons of tumor hap1 to normal hap1 and tumor hap2 to normal hap2, I've uploaded slightly modified tumor haplotype-separated fasta files. You may remember that the chr19_chr22_hap2 chromosome actually has parts of normal chr19_hap1 in addition to parts of chr19_hap2. Specifically, chr19_chr22_hap2:0-8191372 best matches the last ~8Mbp of chr19_hap1. This results in a lot of false somatic calls in this region when you align the respective haplotypes, so we've made alternative fastas that mask chr19_chr22_hap2:0-8191372 on the hap2 fasta and add this sequence as an additional contig to hap1:
hap1: https://nist-giab.s3.us-east-1.amazonaws.com/giab_tumor-normal/analysis/HG008/NIST_asm_dev/HG008T_v3.2/HG008T_v3.2.hap1_withchr19hap2.fasta.gz
hap2: https://nist-giab.s3.us-east-1.amazonaws.com/giab_tumor-normal/analysis/HG008/NIST_asm_dev/HG008T_v3.2/HG008T_v3.2.hap2_maskchr19dup.fasta.gz
These should result in more accurate somatic variants when directly comparing each haplotype

GEt chromosome with noted insertion:
 gunzip -c HG008T_v3.2.hap2_maskchr19dup.fasta.gz | awk '{ if ($0 == ">chr2_chr20_hap2") exit; print }'  | gzip > chr1_chr2_hap2.fasta.gz

Search for L1 plus extra as in figure:
 80516239:ATTATACTCTAAGATATTTCC

Get range: +- 26239
 80490000 - 80542480

Extract range:
 gunzip -c chr1_chr2_hap2.fasta.gz | awk '{ printf("%s",$0) }' |  dd bs=4096 skip=80490000 count=52480 iflag=skip_bytes,count_bytes status=none > l1satROI.fa
add id line for fasta: >l1satROI

Any matches:
 samtools view ../HG008-T.bam | awk '{ printf(">%s_%s_%s_%s\n%s\n",$1,$2,$3,$4,$10) }' | minimap2 -ax map-ont alignL1sat.sh - | awk '{ if ($2 ==4) next; print }' > l1sat.sam

what about normal?
  samtools view ../HG008-N.bam | awk '{ printf(">%s_%s_%s_%s\n%s\n",$1,$2,$3,$4,$10) }' | minimap2 -ax map-ont alignL1sat.sh - | awk '{ if ($2 ==4) next; print }' > l1sat.sam

  samtools view ../HG008-T.bam | awk '{ printf(">%s_%s_%s_%s\n%s\n",$1,$2,$3,$4,$10) }' | minimap2 -ax map-hifi l1satROI.fa - |  awk '{ if ($2 == "4") next; print }' > l1sat.sam
  ont gives a lt more maches but only 10% of read ?
 hifi still gives lots of secondary?
 hifi seesm to run a lot slower too? or maybe the output only show mapped read...


  samtools view ../HG008-T.bam | grep "TTTTTTTTATTATACTCTAAGATATTTCC" | less -S
   samtools view -H  ../HG008-T.bam > header_HG008
  cat header_HG008 motif_search.sam  > h_motif_search.sam
  samtools view -b h_motif_search.sam > h_motif_search.bam
  samtools index h_motif_search.bam

  Reads aligning to chr2 show the break as per Jacob.s figure:
(poly-T not sorted)

<img width="1754" height="852" alt="image" src="https://github.com/user-attachments/assets/925171df-b097-4230-91f5-5a3c0d61d9ca" />

get the reads chr2:
less -S h_motif_search.sam | awk '{ if ($3 == "chr2" ) printf(">%s_%s_%s_%s\n%s\n",$1,$2,$3,$4,$10) }' > chr2_motif_clipped.fa

Lets map these to the tunor ref:


Looking at: LINE-1 insertion intermediates recombine with one another or with DNA breaks to form genome rearrangements

  
