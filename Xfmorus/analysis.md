# Analysis of Xylella fastidiosa subsp. morus

## 1. Create a BLAST database of the X. fastidiosa reference genome.

```
makeblastdb -in /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/AE003849.1.fasta -dbtype nucl
```

## 2. Compare morus strain MulM-D genome with reference genome.
```
blastn -query /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/AXDP01000001.1.fasta -db DB/AE003849.1.fasta -outfmt 6 -out 9a5c-MulMD.crunch
```

## 3. Compare morus strain MUL0034 genome with reference genome.
```
blastn -query /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/CP006740.1.fasta -db DB/AE003849.1.fasta -outfmt 6 -out 9a5c-MUL0034.crunch
```
