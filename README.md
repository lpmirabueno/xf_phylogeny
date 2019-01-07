# *Xylella fastidiosa* (Xf) annotation and orthology search
Pipeline to create a phylogeny using 46 publicly available Xf genomes from GenBank. https://github.com/harrisonlab/Frankia used as guideline.
## 1. Download Xf genomic sequences from GenBank.
The FTP links to the currently 46 publicly available Xf genomes on GenBank (dated 01/2019) are saved in [xf-fna_genbank_links.txt](https://github.com/mirloupa/xf_phylogeny/blob/master/xf-fna_genbank_links.txt). Use the following bash script to download all sequences in the file to your genome directory:
```
for line in $(cat xf_genbank_links.txt); do
  wget $line ~/Xf_proj/Ncbi_46/Genomes
done
```
The following Xanthomonas represenative genomes (FASTA and annotation files also obtained from GenBank) were used as outgroups:
*Xanthomonas campestris* pv. *campestris* str. ATCC 33913
```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/145/GCF_000007145.1_ASM714v1/GCF_000007145.1_ASM714v1_genomic.fna.gz ~/Xf_proj/Ncbi_46/Genomes
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/145/GCF_000007145.1_ASM714v1/GCF_000007145.1_ASM714v1_genomic.gbff.gz ~/Xf_proj/Ncbi_46/Genomes
```
*Xanthomonas oryzae* pv. *oryzae* PXO99A
```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/019/585/GCF_000019585.2_ASM1958v2/GCF_000019585.2_ASM1958v2_genomic.fna.gz ~/Xf_proj/Ncbi_46/Genomes
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/019/585/GCF_000019585.2_ASM1958v2/GCF_000019585.2_ASM1958v2_genomic.gbff.gz ~/Xf_proj/Ncbi_46/Genomes
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
for file in /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/GCA_*; do
  mv /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/$file /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/$(head -1 $file | sed 's/ .*//' | sed 's/>//').fasta
done
```
## 4. Download annotation files (.gbff) from GenBank.
Repeat steps 1 and 2 for this, but instead using the [xf-gbff_genbank_links.txt](https://github.com/mirloupa/xf_phylogeny/blob/master/xf-gbff_genbank_links.txt) file, which contains FTP links to the annotation files.
## 5. Change the annotation file names to their GenBank accession numbers and replace the .gbff extension with .gbk.
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/*.gbff; do
  mv /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/$file /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/$(head -1 $file | tr -s ' ' | cut -d " " -f2).gbk
done
```
## 6. Create a Genus database using Prokka.
```
prokka-genbank_to_fasta_db /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/*.gbk > /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/xf_v2.faa
cd-hit -i /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/xf_v2.faa -o /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/xf_v2 -T 0 -M 0 -g 1 -s 0.8 -c 0.9
rm -fv /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/xf_v2.faa /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/xf_v2.bak.clstr /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/xf_v2.clstr
makeblastdb -dbtype prot -in /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/xf_v2
mkdir /home/mirabl/Xf_proj/Ncbi_44/DB_prokka
mv /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/xf_v2* /home/mirabl/Xf_proj/Ncbi_44/DB_prokka/
```
## 7. Run Prokka and compress (gzip) files.
See https://github.com/tseemann/prokka for documentation.
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/*.fasta; do
  file_short=$(basename $file | sed s/".fasta"//g)
  prokka --usegenus --genus /home/mirabl/Xf_proj/Ncbi_44/DB_prokka/xf_v2 $file --outdir /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/$file_short
  gzip /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/$file
done
```
Move all annotated directories to a new directory named 'Annotation':
```
mv /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/*.1 /home/mirabl/Xf_proj/Ncbi_44/Annotation/
mv /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/*.2 /home/mirabl/Xf_proj/Ncbi_44/Annotation/
```
## 8. Filter genomes based on Levy et al (2018) GWAS paper.
Run quast.py on all FASTA files
```
quast.py /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/*.fasta.gz
```
Filter genomes based on N50 >=40kbp and save only unique genomes into a new file:
```
python /home/hulinm/git_repos/tools/analysis/python_effector_scripts/extract_N50filtered_genomes.py /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/Quast_results/results_2018_12_10_14_08_42/transposed_report.tsv > /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/report2.txt
cut -f1 -d " " /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/report2.txt | uniq > /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/report3.txt 
```
Save reported genomes in a new directory named 'Filtered':
```
mkdir /home/mirabl/Xf_proj/Ncbi_44/Filtered
for file in /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/$(cat report3.txt); do
  cp /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/"$file".fasta /home/mirabl/Xf_proj/Ncbi_44/Filtered/
done
```
## 9. Run CheckM on filtered genomes from step 8.
This script submits the jobs to HPC. CheckM can only be run on blacklace01 or blacklace 06. 
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Filtered/*.fasta ; do
  file_short=$(basename $file | sed s/".fasta"//g) 
  echo $file_short 
  #mkdir -p /home/mirabl/Xf_proj/Ncbi_44/Filtered/Checkm/"$file_short"/Checkm 
  #cp $file /home/mirabl/Xf_proj/Ncbi_44/Filtered/Checkm/"$file_short" 
  Jobs=$(qstat | grep -i 'checkm' | wc -l) 
    while [ $Jobs -gt 5 ]; do 
      sleep 10
      printf "." 
      Jobs=$(qstat | grep -i 'checkm' | wc -l) 
    done
  qsub /home/mirabl/sub_checkm.pbs /home/mirabl/Xf_proj/Ncbi_44/Filtered/Checkm/"$file_short" /home/mirabl/Xf_proj/Ncbi_44/Filtered/Checkm/"$file_short"/Checkm 
done
```
Run CheckM report:
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Filtered/*fasta; do
  file_short=$(basename $file | sed s/".fasta"//g)
  checkm qa /home/mirabl/Xf_proj/Ncbi_44/Filtered/Checkm/"$file_short"/Checkm/lineage.ms /home/mirabl/Xf_proj/Ncbi_44/Filtered/Checkm/"$file_short"/Checkm > Checkm/"$file_short"/Checkm/report
done
```
## 10. Perform orthology analysis on filtered, clean genomes using OrthoFinder.
Rename .faa files (from PROKKA output) to contain genome name not PROKKA output:
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/*.fasta.gz; do
  file_short=$(basename $file | sed s/".fasta.gz"//g)
  echo $file_short
  cp /home/mirabl/Xf_proj/Ncbi_44/Annotation/"$file_short"/*.faa /home/mirabl/Xf_proj/Ncbi_44/Annotation/"$file_short"/"$file_short".faa
done
```
## 11. Copy all .faa files to a new directory named 'Analysis'.
```
mkdir /home/mirabl/Xf_proj/Ncbi_44/Analysis
cp /home/mirabl/Xf_proj/Ncbi_44/Xf_genomes/*/*.faa /home/mirabl/Xf_proj/Ncbi_44/Analysis
```
## 12. Modify all fasta files to remove description, which is the correct format for OrthoMCL.
Each fasta item must be in format of strain|peg.number
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Analysis/*.faa; do
  file_short=$(basename $file | sed s/".faa"//g)
  echo $file_short
  sed 's/ .*//' $file | sed s/"_"/"|peg."/g > /home/mirabl/Xf_proj/Ncbi_44/Analysis/"$file_short".fa
done
```
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Analysis/*.fasta; do
  id=$(less $file | grep ">" | cut -f1 -d "|" | sed s/">"//g | uniq)
  file_short=$(basename $file | sed s/".fa"//g)
  echo $id
  echo $file_short
  sed s/"$id"/"$file_short"/g $file > /home/mirabl/Xf_proj/Ncbi_44/Analysis/$file_short.fasta
done
```
## 13. Remove manually those that did not pass CheckM and also those that did not pass N50 limit and move to new directory OrthoFinder/Formatted
```
mkdir /home/mirabl/Xf_proj/Ncbi_44/OrthoFinder/Formatted
for file in /home/mirabl/Xf_proj/Ncbi_44/Filtered/*.fasta; do
  file_short=$(basename $file | sed s/".fasta"//g | cut -f1,2 -d _ )
  echo $file_short
  mv /home/mirabl/Xf_proj/Ncbi_44/Analysis/$file_short.fasta /home/mirabl/Xf_proj/Ncbi_44/OrthoFinder/Formatted/$file_short.fasta
done
```
## 14. Run OrthoFinder.
Submit to HPC.
```
/home/hulinm/local/src/OrthoFinder-2.2.7_source/orthofinder/orthofinder.py -f Formatted -t 16 -S diamond
```
