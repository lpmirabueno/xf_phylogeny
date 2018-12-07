# *Xylella fastidiosa* (Xf) annotation using Prokka
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
Repeat steps 1 and 2 for this, but instead using the [xf-gbff_genbank_links.txt](https://github.com/mirloupa/xf_phylogeny/blob/master/xf-gbff_genbank_links.txt) file, which contains FTP links to the annotation files.
## 5. Change the annotation file names to their GenBank accession numbers and replace the .gbff extension with .gbk.
```
for file in *.gbff; do mv $file $(head -1 $file | tr -s ' ' | cut -d " " -f2).gbk; done
```
## 6. Create a Genus database using Prokka.
```
prokka-genbank_to_fasta_db *.gbk > xf_v2.faa
cd-hit -i Coccus.faa -o Coccus -T 0 -M 0 -g 1 -s 0.8 -c 0.9
rm -fv xf_v2.faa xf_v2.bak.clstr xf_v2.clstr
makeblastdb -dbtype prot -in xf_v2
mv xf_v2.p* /home/hulinm/local/src/prokka/db/genus/
```
## 7. Run Prokka and compress (gzip) files.
See https://github.com/tseemann/prokka for documentation.
```
for file in *.fasta ; do file_short=$(basename $file | sed s/".fasta"//g) prokka --usegenus --genus xf_v2 $file --outdir $file_short -----force Gzip $file; done
```
## 8. Filter genomes based on Levy et al (2018) GWAS paper.
#### Run quast.py on all FASTA files
```
quast.py *.fasta
```
#### Filter genomes based on N50 >=40kbp and save only unique genomes into a new file:
```
python /home/hulinm/git_repos/tools/analysis/python_effector_scripts/extract_N50filtered_genomes.py quast_results/results/transposed_report.tsv > report2.txt
cut -f1 -d " " report2.txt | uniq > report3.txt 
```

#### Run quast.py on all genomes to get report and save in a new directory named 'Filtered':
```
for file in $(cat report3.txt); do cp "$file".fasta ./Filtered/; done
```
## 9. Run CheckM on filtered genomes from step 8.
This can only run on blacklace01 or blacklace 06. 
```
for file in ./*.fasta ; do
  file_short=$(basename $file | sed s/".fasta"//g) 
  echo $file_short 
  #mkdir -p ./Checkm/"$file_short"/Checkm 
  #cp $file ./Checkm/"$file_short" 
  Jobs=$(qstat | grep -i 'checkm' | wc -l) 
    while [ $Jobs -gt 7 ]; do 
      sleep 10
      printf "." 
      Jobs=$(qstat | grep -i 'checkm' | wc -l) 
    done
  qsub ~/sub_checkm.pbs Checkm/"$file_short" Checkm/"$file_short"/Checkm 
done
```
```
for file in ./*fasta; do
  file_short=$(basename $file | sed s/".fasta"//g)
  checkm qa Checkm/"$file_short"/Checkm/lineage.ms Checkm/"$file_short"/Checkm > Checkm/"$file_short"/Checkm/report
done
```

```
