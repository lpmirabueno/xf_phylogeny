# Effector prediction in _Xylella fastidiosa_

## PREFFECTOR
[PREFFECTOR](http://korkinlab.org/preffector) enables the prediction of bacterial effector proteins across all six secretion systems. Protein sequences in FASTA format are submitted on the web application. The optimised model of choice was selected for all genomes, which allows for fewer false positives. The following pipeline was implemented on the resulting predicted effectors. The PREFFECTOR output format is as follows:
´´´
unique PREFFECTOR identifier, input protein sequence number, minimum predicted probability threshold, actual prediction probability, effector or non-effector classification, FASTA header
´´´

### 1. Filtering of predicted effectors.
Only keep predicted effectors with probability = 0.
´´´
grep ',1,' all_effectors.txt | wc -l
´´´

### 2. Extract sequences of predicted effectors.


### 3. Rename files.


### 4. Modify headers.



### 5. OrthoFinder


### 6. Gain/loss analysis
