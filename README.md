## Xf phylogeny
Pipeline to create a phylogeny using 44 publicly available Xf genomes from GenBank.
# 1. Download Xf genomic sequences from GenBank.
The links to the currently 44 publicly available Xf genomes on GenBank are saved in the xf_genbank_links.txt file. Use the following Bash script to download all sequences in .txt file:
for line in $(cat xf_genbank_links.txt); do wget $line .; done
