# Effector prediction in _Xylella fastidiosa_

## PREFFECTOR
[PREFFECTOR](http://korkinlab.org/preffector) enables the prediction of bacterial effector proteins across all six secretion systems. Protein sequences in FASTA format are submitted on the web application. The optimised model of choice was selected for all genomes, which allows for fewer false positives. The following pipeline was implemented on the resulting predicted effectors. The PREFFECTOR output format is as follows:

unique PREFFECTOR identifier, input protein sequence number, minimum predicted probability threshold, actual prediction probability, effector or non-effector classification, FASTA header

### Filtering and annotation

#### 1. Filtering of predicted effectors.
Only keep predicted effectors with probability = 1.
```
grep ',1,' all_effectors.txt | wc -l
```

#### 2. Create a PROKKA database
```
cd /home/mirabl
prokka-genbank_to_fasta_db/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/SEQ/GFF/*.gff3 > /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/xylella.faa
cd-hit -i xylella.faa -o xylella -T 0 -M 0 -g 1 -s 0.8 -c 0.9
rm -fv xylella.faa xylella.bak.clstr xylella.clstr
makeblastdb -dbtype prot -in data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/xylella
mkdir /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/DB_PROKKA
mv xylella.* /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/DB_PROKKA
```

#### 2. Extract sequences of predicted effectors.
Use NCBI Batch Entrez

#### 3. Rename files to their strain names.
```
for file in *; do
  file_short=$(basename $file | grep "$(sed s/".txt"//g)" xf_strains.csv | cut -d, -f3)
  echo $file_short
  cp "$file" "$file_short".txt
done
```

#### 4. Modify headers to the following format: strain|protein_ID
```
mkdir /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted
for file in /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/SEQ/FAA/*.fasta; do
  strain=$(basename $file | sed s/"_effectors1.fasta"//g);
  echo $strain;
  sed "s/>/>$strain|/" $file | sed 's/ .*//' > /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/"$strain".faa;
done
```
 

#### 5. Run OrthoFinder.
This uses an old version of OrthoFinder and Diamond. First open a new screen session.
```
screen
qlogin
/home/hulinm/local/src/OrthoFinder-2.2.7_source/orthofinder/orthofinder.py -f /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/ -t 16 -S diamond
```

#### 6. Concatenate all protein FASTA files (input from step 5.).
```
cat /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/*.faa > /data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/effector_proteins.fasta
```

#### 7. Extract FASTA sequences for each orthogroup.
```
cd /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28
sed s/"OG"/"orthogroup"/g Orthogroups.csv > Orthogroups2.csv
sed s/"OG"/"orthogroup"/g SingleCopyOrthogroups.txt > SingleCopyOrthogroups2.txt
mkdir Orthogroups/Fasta/
python /home/hulinm/git_repos/tools/pathogen/orthology/orthoMCL/orthoMCLgroups2fasta.py --orthogroups Orthogroups2.csv --fasta /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/effector_proteins.fasta --out_dir Fasta/
#mkdir Fasta/Single_copy
#for file in $(cat SingleCopyOrthogroups2.txt); do
#  echo $file
#  cp Fasta/"$file".faa Fasta/Single_copy
#done
```
Extract orthogroup ID from Orthofinder result.
```
cut -f1 Orthogroups.csv | sed 's/OG/orthogroup//' > Orthogroups_IDs.txt
```

### Alignment.

#### 8. Align the protein sequences of each orthogroup. Submits each orthogroup to HPC.
```
for line in /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/*.fa; do
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

#### 9. Correct alignments using GBlocks.
```
cd /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/
rm Fasta/*.dnd
rm Fasta/*.fa
rm clustalw2.pbs.*
cd ~
for file in /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/*.fasta; do
  Gblocks $file -t=p -d=y
  echo $file
done
```

#### 10. Rename sequences to make them shorter and compatible (change from QTJS01000001.1|peg.00473 to genome name only, i.e. QTJS01000001.1).
```
cd /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/
mkdir Fasta/Align
for fasta in Fasta/*.fasta-gb; do
  name=$(basename $fasta | sed s/".fasta-gb"//g)
  sed '/^>/ s/|.*//' $fasta > Fasta/Align/"$name"
done
```

#### 11. Convert from FASTA to nexus format.
```
cd ~
for file in $(cat /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups_IDs.txt); do
  echo $file
  perl /home/hulinm/git_repos/tools/analysis/python_effector_scripts/alignment_convert.pl -i /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/"$file" -o /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/"$file".nex -f nexus -g fasta
done
```

#### 12. Concatenate single copy orthogroup alignments. Change the path to the input files within the python script.
Cannot be run on screen.
```
cd /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/
python /home/mirabl/SCRIPTS/concatenate.py
```
Open output file on emacs and change datatype to protein.

### Create phylogeny.

#### 13. Convert from nexus to phylip format.
Use http://sequenceconversion.bugaco.com/converter/biology/sequences/nexus_to_phylip.php to convert file

cd /home/mirabl/
perl /home/hulinm/git_repos/tools/analysis/python_effector_scripts/alignment_convert.pl -i /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/combined.nex -o /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/combined.phy -f phylip -g nexus

#### 14. Make partition model file.
```
cd /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/
grep charset combined.nex | sed s/charset//g | sed s/".nex"//g | sed s/"-gb"//g | sed s/" o"/"o"/g | sed s/";"//g > positions
```

#### 15. Order list of genes.
```
cd /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/
cut -f1 -d " " positions > list
```

#### 16. Run the protein model tester on individual alignments.
```
cd /home/mirabl/
for file in $(cat /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups_IDs.txt); do
  echo $file
  perl /home/hulinm/git_repos/tools/analysis/python_effector_scripts/alignment_convert.pl -i /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/"$file" -o /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/"$file".phy -f phylip -g fasta
done
```

#### 17. Test protein models for each orthogroup. 
```
cd /home/mirabl/
for file in /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/*.phy; do
  file_short=$(basename $file | sed s/".phy"//g )
  echo $file_short Jobs=$(qstat | grep 'prottest.p' | wc -l);
    while [ $Jobs -gt 50 ]; do
      sleep 10;
      printf ".";
      Jobs=$(qstat | grep 'prottest.p' | wc -l)
    done
  qsub /home/mirabl/SUB_PBS/Xf_proj/prottest.pbs "$file" /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/"$file_short"_model
done
```

### 27. Get best model name into its own file. ****
```
mkdir /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/Model/
for file in /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/*_model; do
  file_short=$(basename $file | sed s/"_model"//g)
  grep -i "Best model according to LnL" $file | cut -d " " -f6 > /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/Model/"$file_short"
done
```

### 28. Move model files.
```
mkdir /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/Prottest_model
for file in /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/*_model; do
  file_short=$(basename $file | sed s/"_model"//g)
  mv $file /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/Prottest_model/"$file_short"
done
```

### 29. Proteins.
```
for file in $(cat /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/list); do
  cat /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/Model/"$file" >> //data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/Model/models
done
```

### 30. Add sequence evolution model.
Make the final partition file.
```
cd /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/
mkfifo pipe1
mkfifo pipe2
#Add effector names in first column
cut -f1 Fasta/Align/Model/models > pipe1 & cut -f1,2,3 Fasta/Align/positions > pipe2 & paste pipe1 pipe2 > Fasta/Align/partition
mv pipe* /data2/scratch2/mirabl/Discard
sed s/"\t"/", "/g Fasta/Align/partition > Fasta/Align/partition_file
```

### 31. Run RAxML on concatenated protein alignment.
```
cd /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/Model
qsub ~/SUB_PBS/Xf_proj/raxml_partition.pbs /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/combined.phy raxml_cat_aa-aln2.out /data/data2/scratch2/mirabl/Xf_proj/Xf_effector_prediction/Preffector_results_Xf55/Effectors/Probability_1/Analysis/Orthofinder/Formatted/Results_Nov28/Orthogroups/Fasta/Align/partition_file
```

### 32. Run IQTREE.
```
qsub /home/mirabl/SUB_PBS/Xf_proj/iqtree.pbs /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/OrthoFinder/Results_May31/Orthogroups/Fasta/Single_copy/Align/combined.phy /data2/scratch2/mirabl/Xf_proj/NCBI_Xf55/Analysis_w_outgr/OrthoFinder/Formatted/OrthoFinder/Results_May31/Orthogroups/Fasta/Single_copy/Align/partition_file
```

### Gain/loss analysis




