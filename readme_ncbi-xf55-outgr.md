# *Xylella fastidiosa* (Xf) annotation and orthology search
(created 04/2019)  
Pipeline to create a phylogeny using 55 publicly available Xf genomes from GenBank. https://github.com/harrisonlab/Frankia used as guideline.
## 1. Download Xf genomic sequences from GenBank.
The FTP links to the currently 55 publicly available Xf genomes on GenBank (dated 04/2019) are saved in [201903_xf_genbank-ftp_gen-fna.txt]. Use the following bash script to download all sequences in the file to your genome directory:
```
for line in $(cat xf-gbff_genbank_links.txt); do
  wget $line /data2/scratch2/mirabl/Xf_proj/Genomes/Xf/DNA_fasta
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
for file in /data2/scratch2/mirabl/Xf_proj/Genomes/Xf/DNA_fasta/GC*.fna; do
  mv $file /data2/scratch2/mirabl/Xf_proj/Genomes/Xf/DNA_fasta/$(head -1 $file | sed 's/ .*//' | sed 's/>//').fasta
done
for file in ./*.fna; do
  mv $file ./$(head -1 $file | sed 's/ .*//' | sed 's/>//').fasta
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
for file in /data2/scratch2/mirabl/Xf_proj/Genomes/Xf/DNA_gbff/*.gbff; do
  mv $file /data2/scratch2/mirabl/Xf_proj/Genomes/Xf/DNA_gbff/$(head -1 $file | tr -s ' ' | cut -d " " -f2).gbk
done

```
## 6. Create a Genus database using Prokka.
```
cd /home/mirabl/
prokka-genbank_to_fasta_db /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_gbff/*.gbk > /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/xf.faa
cd-hit -i /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/xf.faa -o xf -T 0 -M 0 -g 1 -s 0.8 -c 0.9
mv xf* /data2/scratch2/mirabl/Xf_proj/
rm -fv /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/xf.faa /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/xf.bak.clstr /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/xf.clstr
makeblastdb -dbtype prot -in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/xf
mkdir /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/DB_prokka
mv /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/xf.* /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/DB_prokka/
```
## 7. Run Prokka and compress (gzip) files. Run this in a screen / tmux session.
See https://github.com/tseemann/prokka for documentation.
```
cd /home/mirabl/
for file in /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/*.fasta; do
  file_short=$(basename $file | sed s/".fasta"//g)
  prokka --usegenus --genus /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/xf $file --outdir /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/$file_short
  gzip -f /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/$file
done
```
Move all annotated subdirectories to a new directory named 'Annotation':
```
mkdir /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Annotation/
mv /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/*.1 /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Annotation/
mv /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/*.2 /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Annotation/
```
## 8. Filter genomes based on Levy et al (2018) GWAS paper.
Run quast.py on all FASTA files
```
cd /home/mirabl/
quast.py /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/*.fasta.gz
mv /home/mirabl/quast_results /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/
```
Filter genomes based on N50 >=40kbp and save only unique genomes into a new file:
```
cd /home/mirabl/
python /home/hulinm/git_repos/tools/analysis/python_effector_scripts/extract_N50filtered_genomes.py /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/quast_results/results_2019_04_24_08_52_22/transposed_report.tsv > /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/report2.txt
cut -f1 -d " " /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/report2.txt | uniq > /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/report3.txt 
```
Save reported genomes in a new directory named 'Filtered':
```
mkdir /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered
cd /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/
for file in $(cat report3.txt); do
  cp /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/"$file".fasta.gz Filtered/
done
```
## 9. Run CheckM on filtered genomes from step 8. Run this in a screen / tmux session.
This script submits the jobs to HPC. CheckM can only be run on blacklace01 or blacklace 06. 
```
gunzip /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/*.fasta.gz
mkdir /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/Checkm
for file in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/*.fasta ; do
  file_short=$(basename $file | sed s/".fasta"//g) 
  echo $file_short 
  mkdir -p /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/Checkm/"$file_short"/Checkm 
  cp $file /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/Checkm/"$file_short" 
  Jobs=$(qstat | grep -i 'checkm' | wc -l) 
    while [ $Jobs -gt 5 ]; do 
      sleep 10
      printf "." 
      Jobs=$(qstat | grep -i 'checkm' | wc -l) 
    done
  qsub /home/mirabl/SUB_PBS/Xf_proj/checkm.pbs /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/Checkm/"$file_short" /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/Checkm/"$file_short"/Checkm 
done
```
Run CheckM report:
```
cd /home/mirabl
for file in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/*fasta; do
  file_short=$(basename $file | sed s/".fasta"//g)
  checkm qa /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/Checkm/"$file_short"/Checkm/lineage.ms /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/Checkm/"$file_short"/Checkm > /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/Checkm/"$file_short"/Checkm/report
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
for file in /data2/scratch2/mirabl/Xf_proj/Genomes/All/DNA_fasta/*.fasta.gz; do
  file_short=$(basename $file | sed s/".fasta.gz"//g)
  echo $file_short
  cp /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Annotation/"$file_short"/*.faa /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Annotation/"$file_short"/"$file_short".faa
done
```
## 11. Copy all .faa files to a new directory named 'Analysis'.
```
mkdir /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Analysis
cp /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Annotation/*/*.faa /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Analysis
```
## 12. Modify all fasta files to remove description, which is the correct format for OrthoMCL.
Each fasta item must be in format of strain|peg.number
```
for file in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Analysis/*.faa; do
  file_short=$(basename $file | sed s/".faa"//g)
  echo $file_short
  sed 's/ .*//' $file | sed s/"_"/"|peg."/g > /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Analysis/"$file_short".fa
done
```
```
for file in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Analysis/*.fa; do
  id=$(less $file | grep ">" | cut -f1 -d "|" | sed s/">"//g | uniq)
  file_short=$(basename $file | sed s/".fa"//g)
  echo $id
  echo $file_short
  sed s/"$id"/"$file_short"/g $file > /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Analysis/$file_short.fasta
done
```
## 13. Remove manually those that did not pass CheckM and also those that did not pass N50 limit and move to new directory OrthoFinder/Formatted
```
mkdir /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/ /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted
for file in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Filtered/*.fasta; do
  file_short=$(basename $file | sed s/".fasta"//g | cut -f1,2 -d _ )
  echo $file_short
  mv /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/Analysis/$file_short.fasta /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/$file_short.fasta
done
```
## 14. Run OrthoFinder.
Submit to HPC (change input directory within PBS script).
```
qsub /home/mirabl/SUB_PBS/Xf_proj/orthofinder.pbs
```
## 15. Concatenate all protein FASTA files (output from step 14.).
```
cat /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/*.fasta > /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/proteins.fasta
```
## 16. Extract FASTA sequences for each orthogroup.
```
cd /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/OrthoFinder/Results_May31/Orthogroups/
sed s/"OG"/"orthogroup"/g Orthogroups.txt > Orthogroups2.txt
sed s/"OG"/"orthogroup"/g Orthogroups_SingleCopyOrthologues.txt > Orthogroups_SingleCopyOrthologues2.txt
mkdir Fasta/
python /home/hulinm/git_repos/tools/pathogen/orthology/orthoMCL/orthoMCLgroups2fasta.py --orthogroups Orthogroups2.txt --fasta /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/proteins.fasta --out_dir Fasta/
mkdir Fasta/Single_copy
for file in $(cat Orthogroups_SingleCopyOrthologues2.txt); do
  echo $file
  cp Fasta/"$file".fa Fasta/Single_copy
done
```
## 17. Align the protein sequences of each orthogroup. Submits each orthogroup to HPC.
```
for line in /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/OrthoFinder/Results_May31/Orthogroups/Fasta/Single_copy/*.fa; do
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
## 18. Correct alignments using GBlocks **
```
cd /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/OrthoFinder/Results_May31/Orthogroups/
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
cd /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/OrthoFinder/Results_May31/Orthogroups/
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
## 27. Get best model name into its own file.
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
