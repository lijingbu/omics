# Omics

[IsoSeq3 clustering workflow using conda and GNU-parallel](https://github.com/lijingbu/omics/blob/main/pacbio_isoseq.md)

This workflow process PacBio subreads.bam files, clean barcode primers, refine, and cluster to get high quality transcripts, and finally putout report numbers for each step like below.

[Why GNU-parallel?](https://github.com/lijingbu/omics/blob/main/why_gnu_parallel.md)

With GNU-parallel one can easily transform workflows to run steps in parallel. Jobs run in parallel, and it's easy to change commands, options and number of parallel jobs, no more for loops. 


