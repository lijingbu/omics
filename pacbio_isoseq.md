# IsoSeq3 clustering workflow using conda and GNU-parallel
Refer to the detailed description at https://github.com/PacificBiosciences/IsoSeq/blob/master/isoseq-clustering.md

Required PacBio packages: ccs, lima, isoseq3

### Install conda
A very convenient package managment system for all OS.
Install conda and add bioconda channels.
https://bioconda.github.io/user/install.html

### Install required PacBio packages if not previousely installed
To install GNU-parallel in conda base environment
```conda install -c conda-forge parallel```

`pbccs` package includes `ccs` and ``lima`` package. Here we put the two package into two separate conda environments, in case use them separately in the future.
```
conda create -n pbccs pbccs
conda create -n isoseq3 isoseq3
```

### Create a sample list to store sample names. This list will be used in parallel process below.
```
ls *subreads.bam | sed 's/.subreads.bam//' > list.txt
```
### CLR to CCS
Load pbccs in conda environment
```
conda activate pbccs
```

CCS process utilitize all CPUs detected on the device, it's a CPU heavy step, so run only 1 sample `-j 1` at a time.
```
cat list.txt | parallel -j1 "ccs {}.subreads.bam {}.ccs.bam --min-rq 0.9 --report-file ccs_report{#}.txt >& log.css.s{#}.txt"
```

### Adaptor trimming
Trimming can be done in parallel, note the drop of `-j 1`. Usually 2 SMRT cells, but if too slow, run with `-j N` where `N` is number of parallel jobs to run.
```
cat list.txt | parallel "lima {}.ccs.bam primers.fasta {}.fl.bam --isoseq --peek-guess >& log.lima.s{#}.txt" 
```
To be tidy, run `conda deactivate` to deactivate `pbccs` environment.
### Refine
```
conda activate isoseq3
cat list.txt | parallel "isoseq3 refine --require-polya {}.fl.Clontech_5p--NEB_Clontech_3p.bam primers.fasta {}.flnc.bam >& log.refine.s{#}.txt" 
```

### Merge SMRT cells
```
ls *.flnc.bam  > flnc.fofn
```

### Clustering
```
isoseq3 cluster flnc.fofn clustered.bam --verbose --use-qvs >& log.cluster.txt & 

sed -i 's/.*\r//' log.cluster.txt

zcat clustered.hq.fasta.gz | sed 's|>transcript/|>hq_trans_|; s/ / high_quality /' > clustered.HQ.trans.fasta 
zcat clustered.lq.fasta.gz | sed 's|>transcript/|>lq_trans_|; s/ / low_quality /' > clustered.LQ.trans.fasta
```
To be tidy, run `conda deactivate` to deactivate `isoseq3` environment.

### Report
```
cat list.txt | parallel "echo 'Sample:{#}__{}' > tmp{#}.txt 
head -2 ccs_report{#}.txt >> tmp{#}.txt 
head -2 {}.fl.lima.summary >> tmp{#}.txt 
sed 's/,/\n/g' {}.flnc.filter_summary.json | sed 's/}/\n/; s/[{\"]//g' >> tmp{#}.txt
sed 's/.*://; s/^ //' tmp{#}.txt > num{#}.txt"
cat tmp1.txt | sed 's/:.*//; s/ $//' > rowheader.txt

paste rowheader.txt num*.txt > tmp_report.txt
cat tmp_report.txt | grep -v 'ZMWs input                (A)' | \
 grep -v -P 'num_reads_fl\t' | sed 's/num_reads_flnc\t/Refine num_reads_flnc\t/' | \
 sed 's/ZMWs above all thresholds (B)/ZMWs primer filtered (B)/' \
 > isoseq_report.txt 

head -1 log.cluster.txt | sed 's/Read BAM.* : /-----------\nCluster total\t/; s/(//; s/).*//' >> isoseq_report.txt
zgrep -c '>' clustered.*gz | sed 's/:/\t/' >> isoseq_report.txt
grep -e NumRecords -e TotalLength clustered.transcriptset.xml | sed 's|</.*||; s/.*://; s/>/\t/' >> isoseq_report.txt
cat isoseq_report.txt | column -s $'\t' -t -o $'\t|\t'
```

Report looks like
```
Sample                   | 1__SEL34534_m432423_7575423_34236 | 2__SEL34534_m432423_345435_345576
ZMWs input          (A)  | 668324                            | 652591
ZMWs generating CCS (B)  | 494610 (74.01%)                   | 461709 (70.75%)
ZMWs primer filtered (B) | 436522 (88%)                      | 408402 (88%)
Refine num_reads_flnc    | 430633                            | 403479
num_reads_flnc_polya     | 430414                            | 403291
-----------
Cluster total            | 833705
clustered.hq.fasta.gz    | 54105
clustered.lq.fasta.gz    | 89
TotalLength              | 78596733
NumRecords               | 54194
```
