# Genome Assembly and Annotation Workflow
Notes below were shared for open discussion. Feel free to leave comments (in Issues?) or contact me for questions or suggestions.
Most of the tools can be found by `conda search tool_name` and installed by `conda create -name tool_name tool_name`. Refer here to [Install conda](https://conda.io/projects/conda/en/latest/user-guide/install/index.html). 

## Genome assembly from PacBio HiFi reads (high accuracy long reads): 
1. PacBio HiFi reads assembly [hifiasm](https://github.com/chhylp123/hifiasm) or [HiCanu](https://github.com/marbl/canu).
2. Filter contamination [blobtools2](https://github.com/blobtoolkit/blobtools2). 
3. Scaffolding if long range info data like Hi-C, [Omni-C](https://dovetailgenomics.com/omni-c/), [BioNano Genomics](https://bionanogenomics.com/technology/genome-assembly/).
4. Scaffolds/contigs sort and rename ([GenomeTools](http://genometools.org/), [bedtools](https://bedtools.readthedocs.io/en/latest/)).
5. Evaluation: [assembly-stats](https://github.com/sanger-pathogens/assembly-stats), [QUAST](https://github.com/ablab/quast), [BUSCO](https://busco.ezlab.org/), [Bandage](https://rrwick.github.io/Bandage/).

## Genome assembly from PacBio CLR reads (long but low accuracy reads)
1. CLR reads polishing and assembly ([Canu](https://github.com/marbl/canu)).
2. Illumina PE reads polish ([Pilon](https://github.com/broadinstitute/pilon/wiki)).
3. Scaffolding with Illumina PE reads ([SSPACE](https://github.com/nsoranzo/sspace_basic)).
4. Evaluation: [assembly-stats](https://github.com/sanger-pathogens/assembly-stats), [QUAST](https://github.com/ablab/quast), [BUSCO](https://busco.ezlab.org/).

## Tips for genome assembly evaluation:
After initial assembly and before contamination filtering, the common values to check for genome assembly quality are listed below. After contamination, these numbers may change. 
1. **Genome size** (whether close to expectation or consistent among different assembly methods/options); 
2. **N50** (median like value, the longer the better)
3. **Number of contigs/scaffolds** (smaller is better). 
4. Also the **maximum and minimum length of contig/scaffolds**. These values above could all be calculated quickly by the [assembly-stats](https://github.com/sanger-pathogens/assembly-stats) (-t option gives nice tabular format) listed above. 
5. [BUSCO](https://busco.ezlab.org/) also helps to show if the core orthologs were covered.   
6. Most assemblers will provide assembly graphs e.g. GFA files, which can be viewed by the [Bandage](https://rrwick.github.io/Bandage/).


## Genome annotation
1. Build repeat models from genome assembly [RepeatModeler](http://www.repeatmasker.org/RepeatModeler/) (easy to use in [Dfam-TETools container](https://github.com/Dfam-consortium/TETools)).
2. Filter out repeat models which may be actually protein coding genes ([BLASTx](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download) to close releated species or [InterProScan](https://interproscan-docs.readthedocs.io/en/latest/) domain prediction).
3. Mask genome with clean repeat modes [RepeatMasker](https://www.repeatmasker.org/RepeatMasker/) (easy to use in [Dfam-TETools container](https://github.com/Dfam-consortium/TETools)).
4. Genome annotation using RNA-Seq data + close related proteins ([BRAKER2](https://github.com/Gaius-Augustus/BRAKER)).
5. Genome annotation using close related coding sequences ([PASA](https://github.com/PASApipeline/PASApipeline/wiki)).
6. EvidenceModeler merge predicted gene models from BRAKER2, BUSCO and PASA above.
7. Local host web site for manual checking [Apollo](https://genomearchitect.readthedocs.io/en/latest/).
8. Functional annotation for predicted genes: 
    1. BLASTn to NCBI non-redundant nucleotide database (NT) [NCBI BLAST+ Tools and databases](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download)
    2. BLASTp to NCBI non-redundant protein database (NR, database download see NT above) database and high quality manually curated protein database [Uniprot](https://www.uniprot.org/).
    3. Functional domain predictionfor all predicted proteins ([InterProScan](https://interproscan-docs.readthedocs.io/en/latest/)). 

## **Check out [GenSAS](https://www.gensas.org/), a free online genome annotation server.**
The GenSAS server is free to use without exposing the genome assembly to the public, and it offers functions from RNA-Seq or protein evidence upload to the final Apollo interface for manual correction and sharing. NCBI also offers a genome annotation service [NCBI Eukaryotic Genome Annotation Pipeline](https://www.ncbi.nlm.nih.gov/genome/annotation_euk/process/) for free, but the genome assembly has to be made public before annotation. Also, you may need to write a paragraph of statement (https://www.ncbi.nlm.nih.gov/genome/annotation_euk/policy/) saying why the genome is important to be annotated before it's put into the annotation queue.
