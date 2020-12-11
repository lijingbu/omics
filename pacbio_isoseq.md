### IsoSeq3 workflow using conda and GNU-parallel


# Load pbccs
```
conda activate pbccs
```

# CLR to CCS
```
ls *subreads.bam | sed 's/.subreads.bam//' > list.txt
cat list.txt | parallel -j1 "ccs {}.subreads.bam {}.ccs.bam --min-rq 0.9 --report-file ccs_report{#}.txt >& log.css.s{#}.txt"
```

# Adaptor trimming
```
cat list.txt | parallel "lima {}.ccs.bam primers.fasta {}.fl.bam --isoseq --peek-guess >& log.lima.s{#}.txt" 
```

# Refine
```
conda activate isoseq3
cat list.txt | parallel "isoseq3 refine --require-polya {}.fl.Clontech_5p--NEB_Clontech_3p.bam primers.fasta {}.flnc.bam >& log.refine.s{#}.txt" 
```

# Merge SMRT cells
```
ls *.flnc.bam  > flnc.fofn
```

# Clustering
```
isoseq3 cluster flnc.fofn clustered.bam --verbose --use-qvs >& log.cluster.txt & 

sed -i 's/.*\r//' log.cluster.txt

zcat clustered.hq.fasta.gz | sed 's|>transcript/|>hq_trans_|; s/ / high_quality /' > clustered.HQ.trans.fasta 
zcat clustered.lq.fasta.gz | sed 's|>transcript/|>lq_trans_|; s/ / low_quality /' > clustered.LQ.trans.fasta
```

# report
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

