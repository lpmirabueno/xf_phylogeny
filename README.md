# *Xylella fastidiosa* (Xf) phylogeny
Pipeline to create a phylogeny using 44 publicly available Xf genomes from GenBank.
## 1. Download Xf genomic sequences from GenBank.
The FTP links to the currently 44 publicly available Xf genomes on GenBank (dated 11/2018) are saved in [xf_genbank_links.txt](https://github.com/mirloupa/xf_phylogeny/blob/master/xf_genbank_links.txt). Use the following bash script to download all sequences in the file to your current directory:
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
