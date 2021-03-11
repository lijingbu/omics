# RNA-Seq Experiment design recommendations:

1. **Replicates** is most important for Differential Gene Expression (DGE) if that’s the primary goal. For more statistical power, minimum replicate number is 6 for clinical related study and 4 for basic research. 
2. **Depth** is second. For regular DGE tests, 30M reads per sample is recommenced for human. Mouse/mice are similar. Higher depth is desired for lower expressed genes differential expression test. For alternative splicing events detection, it can be 100M or higher. 
3. Run with one **plate** is known to have unexpected low outputs for some samples. When 2 or more plates is possible, do run all sample in the first plate, then calculate output for each sample. Balance the follow up runs using the first run’s results is turned out much more accurate. Some sample may already reached the targeted depth in the first run, which can be dropped in the future runs. 
4. **Read length**: longer read length ensured higher accuracy for reads mapping to reference, therefore better estimation of expression counts. But it’s not essential if budget is limited. So  150 bp is better and 75bp is fine, as long as replicates and depth is good. 
5. Other **lncRNA or miRNA** experiments may need special library extraction/enrichment method and different consideration of sequence depth. 

## References:

1. Stark, R., Grzelak, M., & Hadfield, J. (2019). RNA sequencing: the teenage years. Nature Reviews Genetics, 20(11), 631–656. https://doi.org/10.1038/s41576-019-0150-2

2. Conesa, A., Madrigal, P., Tarazona, S., Gomez-Cabrero, D., Cervera, A., McPherson, A., Szcześniak, M. W., Gaffney, D. J., Elo, L. L., Zhang, X., & Mortazavi, A. (2016). A survey of best practices for RNA-seq data analysis. Genome Biology, 17(1), 13.https://doi.org/10.1186/s13059-016-0881-8

