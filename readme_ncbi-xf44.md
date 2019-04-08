# *Xylella fastidiosa* (Xf) annotation and orthology search
Pipeline to create a phylogeny using 44 publicly available Xf genomes from GenBank. https://github.com/harrisonlab/Frankia used as guideline.
## 1. Download Xf genomic sequences from GenBank.
The FTP links to the currently 44 publicly available Xf genomes on GenBank (dated 10/2018) are saved in [xf-fna_genbank_links.txt](https://github.com/mirloupa/xf_phylogeny/blob/master/xf-fna_genbank_links.txt). Use the following bash script to download all sequences in the file to your genome directory:
```
for line in $(cat xf-gbff_genbank_links.txt); do
  wget $line /home/mirabl/Xf_proj/Ncbi_44/Genomes
done
```
## 2. Unzip all downloaded sequences.
```
gunzip GC*
```
or
```
gzip -d GC*
```
## 3. Change FASTA file names to their GenBank accession numbers.
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Genomes/GC*.fna; do
  mv $file /home/mirabl/Xf_proj/Ncbi_44/Genomes/$(head -1 $file | sed 's/ .*//' | sed 's/>//').fasta
done
```
## 4. Download annotation files (.gbff) from GenBank.
Repeat steps 1 and 2 for this, but instead using the [xf-gbff_genbank_links.txt](https://github.com/mirloupa/xf_phylogeny/blob/master/xf-gbff_genbank_links.txt) file, which contains FTP links to the annotation files.  
The following Xanthomonas represenative genomes (FASTA and annotation files also obtained from GenBank) were used as outgroups:  
*Xanthomonas campestris* pv. *campestris* str. ATCC 33913
```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/145/GCF_000007145.1_ASM714v1/GCF_000007145.1_ASM714v1_genomic.fna.gz ~/Xf_proj/Ncbi_44/Genomes
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/007/145/GCF_000007145.1_ASM714v1/GCF_000007145.1_ASM714v1_genomic.gbff.gz ~/Xf_proj/Ncbi_44/Genomes
```
*Xanthomonas oryzae* pv. *oryzae* PXO99A
```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/019/585/GCF_000019585.2_ASM1958v2/GCF_000019585.2_ASM1958v2_genomic.fna.gz ~/Xf_proj/Ncbi_44/Genomes
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/019/585/GCF_000019585.2_ASM1958v2/GCF_000019585.2_ASM1958v2_genomic.gbff.gz ~/Xf_proj/Ncbi_44/Genomes
```
## 5. Change the annotation file names to their GenBank accession numbers and replace the .gbff extension with .gbk.
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Genomes/*.gbff; do
  mv $file /home/mirabl/Xf_proj/Ncbi_44/Genomes/$(head -1 $file | tr -s ' ' | cut -d " " -f2).gbk
done
```
## 6. Create a Genus database using Prokka.
```
prokka-genbank_to_fasta_db /home/mirabl/Xf_proj/Ncbi_44/Genomes/*.gbk > /home/mirabl/Xf_proj/Ncbi_46/Genomes/xf.faa
cd-hit -i /home/mirabl/Xf_proj/Ncbi_44/Genomes/xf.faa -o xf -T 0 -M 0 -g 1 -s 0.8 -c 0.9
rm -fv /home/mirabl/Xf_proj/Ncbi_44/Genomes/xf.faa /home/mirabl/Xf_proj/Ncbi_44/Genomes/xf.bak.clstr /home/mirabl/Xf_proj/Ncbi_44/Genomes/xf.clstr
makeblastdb -dbtype prot -in /home/mirabl/Xf_proj/Ncbi_44/Genomes/xf
mkdir /home/mirabl/Xf_proj/Ncbi_44/DB_prokka
mv /home/mirabl/Xf_proj/Ncbi_44/Genomes/xf.* /home/mirabl/Xf_proj/Ncbi_44/DB_prokka/
mv /home/mirabl/Xf_proj/Ncbi_44/Genomes/xf /home/mirabl/Xf_proj/Ncbi_44/DB_prokka/
```
## 7. Run Prokka and compress (gzip) files. Run this in a screen / tmux session.
See https://github.com/tseemann/prokka for documentation.
```
for file in /home/mirabl/Xf_proj/Ncbi_44/Genomes/*.fasta; do
  file_short=$(basename $file | sed s/".fasta"//g)
  prokka --usegenus --genus /home/mirabl/Xf_proj/Ncbi_44/DB_prokka/xf $file --outdir /home/mirabl/Xf_proj/Ncbi_44/Genomes/$file_short
  gzip /home/mirabl/Xf_proj/Ncbi_44/Genomes/$file
done
```
Move all annotated subdirectories to a new directory named 'Annotation':
```
mkdir /home/mirabl/Xf_proj/Ncbi_44/Annotation/
mv /home/mirabl/Xf_proj/Ncbi_44/Genomes/*.1 /home/mirabl/Xf_proj/Ncbi_44/Annotation/
mv /home/mirabl/Xf_proj/Ncbi_44/Genomes/*.2 /home/mirabl/Xf_proj/Ncbi_44/Annotation/
```
## 8. Filter genomes based on Levy et al (2018) GWAS paper.
Run quast.py on all FASTA files
```
quast.py /home/mirabl/Xf_proj/Ncbi_445/Genomes/*.fasta.gz
```
Filter genomes based on N50 >=40kbp and save only unique genomes into a new file:
```
python /home/hulinm/git_repos/tools/analysis/python_effector_scripts/extract_N50filtered_genomes.py /home/mirabl/Xf_proj/Ncbi_44/Genomes/quast_results/latest/transposed_report.tsv > /home/mirabl/Xf_proj/Ncbi_44/Genomes/report2.txt
cut -f1 -d " " /home/mirabl/Xf_proj/Ncbi_44/Genomes/report2.txt | uniq > /home/mirabl/Xf_proj/Ncbi_44/Genomes/report3.txt 
```
Save reported genomes in a new directory named 'Filtered':
```
mkdir /home/mirabl/Xf_proj/Ncbi_44/Filtered
for file in /home/mirabl/Xf_proj/Ncbi_44/Genomes/$(cat report3.txt); do
  cp "$file".fasta.gz /home/mirabl/Xf_proj/Ncbi_44/Filtered/
done
```
## 9. Run CheckM on filtered genomes from step 8. Run this in a screen / tmux session.
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
  qsub /home/mirabl/SUB_PBS/Xf_proj/checkm.pbs /home/mirabl/Xf_proj/Ncbi_44/Filtered/Checkm/"$file_short" /home/mirabl/Xf_proj/Ncbi_44/Filtered/Checkm/"$file_short"/Checkm 
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
for file in /home/mirabl/Xf_proj/Ncbi_44/Genomes/*.fasta.gz; do
  file_short=$(basename $file | sed s/".fasta.gz"//g)
  echo $file_short
  cp /home/mirabl/Xf_proj/Ncbi_44/Annotation/"$file_short"/*.faa /home/mirabl/Xf_proj/Ncbi_44/Annotation/"$file_short"/"$file_short".faa
done
```
## 11. Copy all .faa files to a new directory named 'Analysis'.
```
mkdir /home/mirabl/Xf_proj/Ncbi_44/Analysis
cp /home/mirabl/Xf_proj/Ncbi_44/Annotation/*/*.faa /home/mirabl/Xf_proj/Ncbi_44/Analysis
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
for file in /home/mirabl/Xf_proj/Ncbi_44/Analysis/*.fa; do
  id=$(less $file | grep ">" | cut -f1 -d "|" | sed s/">"//g | uniq)
  file_short=$(basename $file | sed s/".fa"//g)
  echo $id
  echo $file_short
  sed s/"$id"/"$file_short"/g $file > /home/mirabl/Xf_proj/Ncbi_44/Analysis/$file_short.fasta
done
```
## 13. Remove manually those that did not pass CheckM and also those that did not pass N50 limit and move to new directory OrthoFinder/Formatted
```
mkdir /home/mirabl/Xf_proj/Ncbi_44/OrthoFinder/
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
qsub /home/mirabl/SUB_PBS/Xf_proj/orthofinder.pbs
```
