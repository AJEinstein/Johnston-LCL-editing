# TBC1D4 Ampilcon-seq analysis 
## Andrew D. Johnston
### 06/14/2018


I targeted Cas9 with a gRNAs 3A to the TBC1D4 intronic enhancer. Additionaly, I also supplied a 3A guided Cas9 with a ssODN containing a three separate bp edits. 

Therefore, I have two samples:

1. 3A
2. 3A + ssODN

The samples were sequenced on MiSeq 250bp-single-end run (I believe that a 15% PhiX spike in was used). 

In addition to trimming the illumina adaptor sequences and poor quality bases. The 5' trimming argument was added to account for the 2 Ns in the FWD locus specific primer. 

```bash
 #clip 2 bases off these at beginning
qsub -cwd -q all.q@@penguin -N di_clipTrim-3A-2  -j y -S /bin/bash -l h_vmem=20G -b y "module load trim_galore/0.3.7/gcc.4.4.7; module list; trim_galore --adapter AGATCGGAAGAGC --clip_R1 2 3A-2.TTGTTG.J872.BV7KB.L1.1.fastq.gz;"
qsub -cwd -q all.q@@penguin -N di_clipTrim-3AO-2 -j y -S /bin/bash -l h_vmem=20G -b y "module load trim_galore/0.3.7/gcc.4.4.7; module list; trim_galore --adapter AGATCGGAAGAGC --clip_R1 2 3AO-2.AACAAC.J872.BV7KB.L1.1.fastq.gz;"
```

Next I downsampled the reads to 500k only (except 4g which was below this theshold)

```bash
cd /gs/gsfs0/users/anjohnst/Programs/bbmap
module load java/1.8.0_20
./reformat.sh in1=~/17_member/TBC1D4/AmpliconSeq/Rep2/fastq/clippedAndAdapterTrimmed_3A-2.TTGTTG.J872.BV7KB.L1.1_trimmed.fq.gz out1=~/17_member/TBC1D4/AmpliconSeq/Rep2/fastq/3A-amp_DS500k.fastq.gz reads=1000000 samplerate=0.5	

./reformat.sh in1=~/17_member/TBC1D4/AmpliconSeq/Rep2/fastq/clippedAndAdapterTrimmed_3AO-2.AACAAC.J872.BV7KB.L1.1_trimmed.fq.gz out1=~/17_member/TBC1D4/AmpliconSeq/Rep2/fastq/3AO-amp_DS500k.fastq.gz reads=1000000 samplerate=0.5
```


Now to run CRISPResso on the downsampled bams for the TBC1D4 locus
Here is the amplicon for the TBC1D4 locus:
```
CAAGGAAAATAAAGGGTCAAGTCAAtttcaaataaattcccctctaaaat
atacctctaggtataaaattaccacagagacacttgagctggcaatgtga
gtcctgcatcactaaaaggagagttctatacacagaaacaaatccgtctt
cacatcaaagctgtctatattggtatgggcatttttctagggccacaaaa
atgaagggggatgttagcttccttgtgaaatgattactcatgtcatttag
aacttgcaagagtgccaggttttaaaatgttttctcccacatatgcttaa
aagtgatttttttttctaggaaccaataagattaattCTGCAGAGCCAGT
AAGAGGAAG
```

Now to run it:
```bash
cd /gs/gsfs0/users/anjohnst/17_member/TBC1D4/AmpliconSeq/Rep2/fastq/
mkdir ../CRISPResso_500k
mv *DS* ../CRISPResso_500k/
mv fastq/clippedAndAdapterTrimmed_4g-2.AACCGA.J872.BV7KB.L1.1_trimmed.fq.gz ../CRISPResso_500k/
cd ../CRISPResso_500k

qsub -S /bin/bash -N gRNA_3A -cwd -R y -l h_vmem=100G -j y -q highmem.q << EOF
module load CRISPResso
source activate CRISPResso
CRISPResso -r1 3A-amp_DS500k.fastq.gz -a CAAGGAAAATAAAGGGTCAAGTCAAtttcaaataaattcccctctaaaatatacctctaggtataaaattaccacagagacacttgagctggcaatgtgagtcctgcatcactaaaaggagagttctatacacagaaacaaatccgtcttcacatcaaagctgtctatattggtatgggactttcctagggccacaaaaatgaagggggatgttagcttccttgtgaaatgattactcatgtcattt -q 30 -s 20 --min_identity_score 60.0 -n 3A_500k -g ATTGGTATGGGACTTTCCTA -w 16 --exclude_bp_from_left 25
EOF

qsub -S /bin/bash -N gRNA_3AO_500k -cwd -R y -l h_vmem=100G -j y -q highmem.q << EOF
module load CRISPResso
source activate CRISPResso
CRISPResso -r1 3AO-amp_DS500k.fastq.gz -a CAAGGAAAATAAAGGGTCAAGTCAAtttcaaataaattcccctctaaaatatacctctaggtataaaattaccacagagacacttgagctggcaatgtgagtcctgcatcactaaaaggagagttctatacacagaaacaaatccgtcttcacatcaaagctgtctatattggtatgggactttcctagggccacaaaaatgaagggggatgttagcttccttgtgaaatgattactcatgtcattt -e CAAGGAAAATAAAGGGTCAAGTCAAtttcaaataaattcccctctaaaatatacctctaggtataaaattaccacagagacacttgagctggcaatgtgagtcctgcatcactaaaaggagagttctatacacagaaacaaatccgtcttcacatcaaagctgtctatattggtatgggcatttttctagggccacaaaaatgaagggggatgttagcttccttgtgaaatgattactcatgtcattt -q 30 -d gggcatttttc -s 20 --min_identity_score 60.0 -n 3AO_HDR_500k -g ATTGGTATGGGACTTTCCTA --hdr_perfect_alignment_threshold 100.0 -w 16 --exclude_bp_from_left 25 --exclude_bp_from_right 0 --keep_intermediate
EOF
```

