# *Xylella fastidiosa* (Xf) phylogeny
Pipeline to create a phylogeny using 44 publicly available Xf genomes from GenBank.
## 1. Download Xf genomic sequences from GenBank.
The links to the currently 44 publicly available Xf genomes on GenBank (dated 11/2018) are saved in the xf_genbank_links.txt file. Use the following bash script to download all sequences in .txt file to your current directory:
```
for line in $(cat xf_genbank_links.txt); do wget $line .; done
```
## 2. Unzip all downloaded sequences.
```
gunzip GCA_*
```
or
```
gzip -d GCA_*
```
## 3. Change FASTA file names to their GenBank accession numbers.
