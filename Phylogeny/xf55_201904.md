# *Xylella fastidiosa* (Xf) annotation and orthology search
(created 04/2019)  
Pipeline to create a phylogeny using 55 publicly available Xf genomes from GenBank. https://github.com/harrisonlab/Frankia used as guideline.
## 1. Download Xf genomic sequences from GenBank.
The FTP links to the currently 55 publicly available Xf genomes on GenBank (dated 04/2019) are saved in [201903_xf_genbank-ftp_gen-fna.txt]. Use the following bash script to download all sequences in the file to your genome directory:
```
for line in $(cat xf-gbff_genbank_links.txt); do
  wget $line /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/
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
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/GC*.fna; do
  mv $file /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/$(head -1 $file | sed 's/ .*//' | sed 's/>//').fasta
done
```
## 4. Download annotation files (.gbff) from GenBank.
Repeat steps 1 and 2 for this, but instead using the [201903_xf_genbank-ftp_gen-gbff.txt] file, which contains FTP links to the annotation files.  
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
## 5. Change the annotation file names to their GenBank accession numbers and replace the .gbff extension with .gbk.
```
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_gbff/*.gbff; do
  mv $file /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_gbff/$(head -1 $file | tr -s ' ' | cut -d " " -f2).gbk
done
```
## 6. Create a Genus database using Prokka.
```
prokka-genbank_to_fasta_db /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_gbff/*.gbk > /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/xf.faa
cd-hit -i /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/xf.faa -o xf -T 0 -M 0 -g 1 -s 0.8 -c 0.9
rm -fv /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/xf.faa /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/xf.bak.clstr /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/xf.clstr
makeblastdb -dbtype prot -in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/xf
mkdir /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DB_prokka
mv /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DB_prokka/
mv /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DB_prokka/
```
## 7. Run Prokka and compress (gzip) files. Run this in a screen / tmux session.
See https://github.com/tseemann/prokka for documentation.
```
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/*.fasta; do
  file_short=$(basename $file | sed s/".fasta"//g)
  prokka --usegenus --genus //home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DB_prokka/xf $file --outdir /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/$file_short
  gzip /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/$file
done
```
Move all annotated subdirectories to a new directory named 'Annotation':
```
mkdir /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Annotation/
mv /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Annotation/
mv /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Annotation/
```
## 8. Filter genomes based on Levy et al (2018) GWAS paper.
Run quast.py on all FASTA files
```
quast.py /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/*.fasta.gz
```
Filter genomes based on N50 >=40kbp and save only unique genomes into a new file:
```
python /home/hulinm/git_repos/tools/analysis/python_effector_scripts/extract_N50filtered_genomes.py /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/quast_results/latest/transposed_report.tsv > /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/report2.txt
cut -f1 -d " " /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/report2.txt | uniq > /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/report3.txt 
```
Save reported genomes in a new directory named 'Filtered':
```
mkdir /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/$(cat report3.txt); do
  cp "$file".fasta.gz /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/
done
```
## 9. Run CheckM on filtered genomes from step 8. Run this in a screen / tmux session.
This script submits the jobs to HPC. CheckM can only be run on blacklace01 or blacklace 06. 
```
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/*.fasta ; do
  file_short=$(basename $file | sed s/".fasta"//g) 
  echo $file_short 
  #mkdir -p /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/Checkm/"$file_short"/Checkm 
  #cp $file /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/Checkm/"$file_short" 
  Jobs=$(qstat | grep -i 'checkm' | wc -l) 
    while [ $Jobs -gt 5 ]; do 
      sleep 10
      printf "." 
      Jobs=$(qstat | grep -i 'checkm' | wc -l) 
    done
  qsub /home/mirabl/SUB_PBS/Xf_proj/checkm.pbs /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/Checkm/"$file_short" /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/Checkm/"$file_short"/Checkm 
done
```
Run CheckM report:
```
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/*fasta; do
  file_short=$(basename $file | sed s/".fasta"//g)
  checkm qa /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/Checkm/"$file_short"/Checkm/lineage.ms /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/Checkm/"$file_short"/Checkm > Checkm/"$file_short"/Checkm/report
done
```
Append CheckM report for each genome into a single summary file "checkm_report"
```
cat AAAL02000032.1/Checkm/report > checkm_report
for file in ./*; do
  file_short=$(basename $file)
  echo $file_short
  cat "$file_short"/Checkm/report | tail -2 >> checkm_report
done
```
## 10. Perform orthology analysis on filtered, clean genomes using OrthoFinder.
Rename .faa files (from PROKKA output) to contain genome name not PROKKA output:
```
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/DNA_fasta/*.fasta.gz; do
  file_short=$(basename $file | sed s/".fasta.gz"//g)
  echo $file_short
  cp /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Annotation/"$file_short"/*.faa /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Annotation/"$file_short"/"$file_short".faa
done
```
## 11. Copy all .faa files to a new directory named 'Analysis'.
```
mkdir /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Analysis
cp /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Annotation/*/*.faa /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Analysis
```
## 12. Modify all fasta files to remove description, which is the correct format for OrthoMCL.
Each fasta item must be in format of strain|peg.number
```
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Analysis/*.faa; do
  file_short=$(basename $file | sed s/".faa"//g)
  echo $file_short
  sed 's/ .*//' $file | sed s/"_"/"|peg."/g > /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Analysis/"$file_short".fa
done
```
```
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Analysis/*.fa; do
  id=$(less $file | grep ">" | cut -f1 -d "|" | sed s/">"//g | uniq)
  file_short=$(basename $file | sed s/".fa"//g)
  echo $id
  echo $file_short
  sed s/"$id"/"$file_short"/g $file > /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Analysis/$file_short.fasta
done
```
## 13. Remove manually those that did not pass CheckM and also those that did not pass N50 limit and move to new directory OrthoFinder/Formatted
```
mkdir /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/
mkdir /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted
for file in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Xf/Filtered/*.fasta; do
  file_short=$(basename $file | sed s/".fasta"//g | cut -f1,2 -d _ )
  echo $file_short
  mv /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/Analysis/$file_short.fasta /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/$file_short.fasta
done
```
## 14. Run OrthoFinder.
Submit to HPC.
```
qsub /home/mirabl/SUB_PBS/Xf_proj/orthofinder.pbs
```
## 15. Concatenate all protein FASTA files (output from step 14.).
```
cat /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/*.fasta > /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/proteins.fasta
```
## 16. Extract FASTA sequences for each orthogroup.
```
cd /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/
sed s/"OG"/"orthogroup"/g Orthogroups.txt > Orthogroups2.txt
sed s/"OG"/"orthogroup"/g SingleCopyOrthogroups.txt > SingleCopyOrthogroups2.txt
mkdir Fasta/
python /home/hulinm/git_repos/tools/pathogen/orthology/orthoMCL/orthoMCLgroups2fasta.py --orthogroups Orthogroups2.txt --fasta /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/proteins.fasta --out_dir Fasta/
mkdir Fasta/Single_copy
for file in $(cat SingleCopyOrthogroups2.txt); do
  echo $file
  cp Fasta/"$file".fa Fasta/Single_copy
done
```
## 17. Align the protein sequences of each orthogroup. Submits each orthogroup to HPC.
```
for line in /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/*.fa; do
  file_short=$(basename $line | sed s/".fa"//g)
  Jobs=$(qstat | grep 'clustalw2' | wc -l)
    while [ $Jobs -gt 100 ]; do
      sleep 10
      printf "."
      Jobs=$(qstat | grep 'clustalw2' | wc -l)
    done
  qsub /home/mirabl/SUB_PBS/Xf_proj/clustalw2.pbs $line
done
```
## 18. Correct alignments using GBlocks 
```
cd /home/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/
rm Fasta/Single_copy/*.dnd
rm Fasta/Single_copy/*.fa
rm clustalw2.pbs*
for file in Fasta/Single_copy/*.fasta; do
  Gblocks $file -t=p -d=y
  echo $file
done
```
## 19. Rename sequences to make them shorter and compatible (change from QTJS01000001.1|peg.00473 to genome name only, i.e. QTJS01000001.1)
```
cd /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/
mkdir Fasta/Single_copy/Align
for fasta in Fasta/Single_copy/*.fasta-gb; do
  name=$(basename $fasta | sed s/".fasta-gb"//g)
  sed '/^>/ s/|.*//' $fasta > Fasta/Single_copy/Align/"$name"
done

```
## 20. Convert from fasta to nexus format.
```
cd ~
for file in $(cat /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/SingleCopyOrthogroups2.txt); do
  echo $file
  perl /home/hulinm/git_repos/tools/analysis/python_effector_scripts/alignment_convert.pl -i /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/"$file" -o /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/"$file" .nex -f nexus -g fasta
done
```
## 21. Concatenate single copy orthogroup alignments. Change the path to the input files within the perl script.
```
python /home/mirabl/SCRIPTS/concatenate.py
```
Use emacs to change datatype to protein.

## 22. Convert from nexus to phylip format.
```
cd /home/mirabl/
perl /home/hulinm/git_repos/tools/analysis/python_effector_scripts/alignment_convert.pl -i /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/combined.nex -o /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/combined.phy -f phylip -g nexus
```
## 23. Make partition model file.
```
grep charset combined.nex | sed s/charset//g | sed s/".nex"//g | sed s/"-gb"//g | sed s/" o"/"o"/g | sed s/";"//g > positions
```
## 24. Order list of genes.
```
cut -f1 -d " " positions > list
```
## 25. Run the protein model tester on individual alignments.
```
cd /home/mirabl/
for file in $(cat /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/SingleCopyOrthogroups2.txt); do
  echo $file
  perl /home/hulinm/git_repos/tools/analysis/python_effector_scripts/alignment_convert.pl -i /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/"$file" -o /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/"$file".phy -f phylip -g fasta
done
```
## 26. Test protein models for each orthogroup.
```
cd /home/mirabl/
for file in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/*.phy; do
  file_short=$(basename $file | sed s/".phy"//g )
  echo $file_short Jobs=$(qstat | grep 'prottest.p' | wc -l)
    while [ $Jobs -gt 50 ]; do
      sleep 10
      printf "."
      Jobs=$(qstat | grep 'prottest.p' | wc -l)
    done
  qsub /home/mirabl/SUB_PBS/Xf_proj/prottest.pbs "$file" /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/"$file_short"_model
done
```
## 27. Get best model name into its own file. **
```
mkdir /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/Model/
for file in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/*_model; do
  file_short=$(basename $file | sed s/"_model"//g)
  grep -i "Best model according to LnL" $file | cut -d " " -f6 > /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/Model/"$file_short"
done
```

## 28. Move model files.
```
mkdir /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/Prottest_model
for file in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/*_model; do
  file_short=$(basename $file | sed s/"_model"//g)
  mv $file /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/Prottest_model/"$file_short"
done
```
## 29. Proteins
```
for file in $(cat /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/list); do
  cat /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/Model/"$file" >> /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/Model/models
done
```
## 30. Add sequence evolution model.
Make the final partition file.
```
cd /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08
mkfifo pipe1
mkfifo pipe2
#Add effector names in first column
cut -f1 Fasta/Single_copy/Align/Model/models > pipe1 & 
cut -f1,2,3 Fasta/Single_copy/Align/positions > pipe2 &
paste pipe1 pipe2 > Fasta/Single_copy/Align/partition
mv pipe* /data2/scratch2/mirabl/Discard
sed s/"\t"/", "/g Fasta/Single_copy/Align/partition > Fasta/Single_copy/Align/partition_file
```
## 31. Run RAxML on concatenated protein alignment.
```
qsub /home/hulinm/git_repos/pseudomonas/orthomcl/sub_raxml_partition_aa.sh combined.phy output partition_file
```
## 32. Run IQTREE.
```
qsub /home/mirabl/SUB_PBS/Xf_proj/iqtree.pbs /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/combined.phy /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Genome_seq/OrthoFinder/Formatted/Results_Apr08/Fasta/Single_copy/Align/partition_file
```
## 33. Blast nod genes.
```
for GENOME in /home/hulinm/frankia/genomes/*.fna; do
  GENOME_FILE=$(basename $GENOME)
  GENOME_SHORT=$(echo $GENOME_FILE | sed s/.fna//g)
  echo $GENOME_SHORT
  python /home/hulinm/git_repos/tools/analysis/python_effector_scripts/rename.py -i "$GENOME_SHORT".fna -o "$GENOME_SHORT".fa
  gzip "$GENOME_SHORT".fna
done

for genome in /home/hulinm/frankia/genomes/*.fa; do
  echo $genome
  file=$(basename $genome)
  genome_short=$(echo $file | sed s/.fa//g) echo $genome_short
    for query in /home/hulinm/frankia/nod_genes/*.fa; do
      echo $query
      query_short=$(basename $query | sed s/.fa//g)
      /home/hulinm/git_repos/tools/pathogen/blast/blast2csv.pl $query tblastn $genome 5 > /home/hulinm/frankia/nod_genes/blast/"$genome_short"_"$query_short"
    done
done
```
