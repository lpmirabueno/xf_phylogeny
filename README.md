# *Xylella fastidiosa* (Xf) phylogeny
Pipeline to create a phylogeny using 44 publicly available Xf genomes from GenBank.
## 1. Download Xf genomic sequences from GenBank.
The FTP links to the currently 44 publicly available Xf genomes on GenBank (dated 11/2018) are saved in [xf-fna_genbank_links.txt](https://github.com/mirloupa/xf_phylogeny/blob/master/xf-fna_genbank_links.txt). Use the following bash script to download all sequences in the file to your current directory:
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
```
for file in GCA_*; do mv $file $(head -1 $file | sed 's/ .*//' | sed 's/>//').fasta; done
```
## 4. Download annotation files (.gbff) from GenBank.
Repeat steps 1. and 2. for this, but instead using the [xf-gbff_genbank_links.txt](https://github.com/mirloupa/xf_phylogeny/blob/master/xf-gbff_genbank_links.txt) file, which contains FTP links to the annotation files.
## 5. Change the annotation file names to their GenBank accession numbers and replace the .gbff extension with .gbk.
```
for file in *.gbff; do mv $file $(head -1 $file | tr -s ' ' | cut -d " " -f2).gbk; done
```
## 6. Create a Genus database using Prokka (see https://github.com/tseemann/prokka for documentation)
```
prokka-genbank_to_fasta_db *.gbk > xf_v2.faa
cd-hit -i Coccus.faa -o Coccus -T 0 -M 0 -g 1 -s 0.8 -c 0.9
rm -fv xf_v2.faa xf_v2.bak.clstr xf_v2.clstr
makeblastdb -dbtype prot -in xf_v2
mv xf_v2.p* /home/hulinm/local/src/prokka/db/genus/
```
