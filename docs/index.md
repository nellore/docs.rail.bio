## Quick start

[Installation](installation.md) tells you how to install Rail-RNA. You can then figure out how to use it by trial and error or try the [Tutorial](tutorial.md). [Deliverables](deliverables.md) describes Rail's various outputs and how to suppress ones you don't need to save time and money. [Reference](reference.md) covers command-line parameters across all of Rail's modes and job flows.

## What is Rail-RNA?

Rail-RNA is a spliced alignment program: it performs intron-aware alignment of RNA sequencing (RNA-seq) data to a genome assembly. You may have heard it's designed to run in the cloud with [Amazon Web Services](http://aws.amazon.com/) [Elastic MapReduce](http://aws.amazon.com/elasticmapreduce/), and you may wonder why it exists since there are already many other spliced alignment programs, so we start with an

## Advertisement

* Rail-RNA doesn't just work in the cloud; it also works on computer clusters using batch schedulers like PBS or SGE via [IPython Parallel](http://ipython.org/ipython-doc/dev/parallel/) and on single computers. If ever you see Rail described as cloud-based software, don't think, "Then it's not for me because I'm never going to use a cloud computing service":

    1. Rail-RNA is for almost everyone working with RNA-seq data. We observe great performance on Illumina HiSeq/MiSeq data, aligning <= 300-bp reads to human-size genomes. Our case is laid out in [this paper](http://biorxiv.org/content/early/2015/05/07/019067).

    2. Never say never. We've tried to make deploying Rail on Amazon Web Services easy enough to convince you to give the cloud a chance for analyzing a large RNA-seq dataset, and we're actively working on improving user experience. Think [freedom through the cloud](http://cloudman.irb.hr/) rather than [freedom from the cloud](https://liorpachter.wordpress.com/2015/05/10/near-optimal-rna-seq-quantification-with-kallisto/): while the latter attitude has its merits, with the former, you won't need to compete for compute with other users of your institutional cluster, and the extent of your analyses won't be constrained by hard upper bounds on disk space and number of processing cores.

    3. Do you use [a streaming music service](https://en.wikipedia.org/wiki/Beats_Music), [Netflix](http://www.amazon.com/Prime-Instant-Video/b?node=2676882011), [webmail](https://en.wikipedia.org/wiki/Webmail#Early_implementations), or [Dropbox](https://dl.dropboxusercontent.com/u/27532820/app.html)? Then you can thank cloud computing for reducing processing and storage burden on your local devices. Why wouldn't you try it out for your bioinformatics too? You already use the [UCSC Genome Browser](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC2834533/).

* Rail-RNA is [(mostly)](https://github.com/nellore/rail/blob/master/README.md) MIT-licensed; you can and are encouraged to hack away at it. In fact, we've tried to make repackaging Rail really easy for you: you can [roll your own installer](installation.md#rolling-your-own-installer). The meat of the source is a bunch of Python scripts, and you can view them [here](https://github.com/nellore/rail/tree/master/src). In particular, check out the scripts for the steps Rail-RNA executes [here](https://github.com/nellore/rail/tree/master/src)). Each one has a long docstring explaining the formats of its input and output. Perhaps you'll want to mess with the step scripts to tweak outputs, or you'll submit us a [pull request](https://github.com/nellore/rail/pulls) to fix a bug. (Do that, please.) Our [paper](http://biorxiv.org/content/early/2015/05/07/019067) and especially its supplementary material explain in detail the algorithms used by each step.

* Rail-RNA isn't just another aligner. While many spliced alignment programs output only SAM or BAM and perhaps a couple more text files summarizing intronic content for a single sample, Rail-RNA has several more outputs:

    1. [bigWig](http://genome.ucsc.edu/goldenpath/help/bigWig.html) files encoding coverage of the genome by reads in `coverage_bigwigs/`. For each sample analyzed, there's one bigWig storing genome coverage by primary alignments and another storing genome coverage by uniquely aligning reads. There are also bigWigs with mean and median genome coverages across samples. To normalize each sample's contribution to these measures of center, the number of reads covering a given base of the genome is multiplied by some library size parameter (by default 40 million) and divided by the number of mapped reads in the sample. A bigWig file is an order of magnitude smaller than a BAM file---about 100 MB for 50 million reads. It can be used as input to several downstream analysis packages, including [`derfinder`](http://bioconductor.org/packages/release/bioc/html/derfinder.html), [`edgeR-robust`](http://bioconductor.org/packages/release/bioc/html/edgeR.html), and [`DESeq2`](http://bioconductor.org/packages/release/bioc/html/DESeq.html). (See Leo Collado-Torres's explanation of how [here](http://lcolladotor.github.io/protocols/bigwig_DEanalysis/).) In fact, Rail's bigWig output is designed to work especially well with `derfinder`, which identifies differentially expressed regions of the genome among groups of samples. The Rail+`derfinder` pipeline is fully agnostic to gene annotation. Studying expression this way increases potential for discovery: in [our Rail+`derfinder` reanalysis](http://biorxiv.org/content/early/2015/05/07/019067) of the [GEUVADIS dataset](http://www.geuvadis.org) comprised of 667 paired-end lymphoblastoid cell line samples, 7.3% of expressed regions we uncovered occurred outside annotated genes.

    2. feature matrices stored as gzipped TSV files (`cross_sample_outputs/introns.tsv.gz`, `cross_sample_outputs/insertions.tsv.gz`, and `cross_sample_outputs/deletions.tsv.gz`. The (i, j)th element of a given matrix is the number of reads in sample j in which evidence of feature i was found, where a feature is an intron, insertion, or deletion. A given feature matrix is straightforward to load in `R` with [`read.table`](https://stat.ethz.ch/R-manual/R-devel/library/utils/html/read.table.html).

    3. a read count matrix `cross_sample_outputs/read_counts.tsv`. The (i, j)th element of this matrix contains the number of primary alignments of reads in sample i to chromosome j and the number of those alignments that are unique separated by a comma. Here, "unique" means that only one alignment of a given read was found to have the highest alignment score. The read count matrix is useful for normalization and required by `derfinder`. A supplementary normalization matrix `cross_sample_outputs/normalization.tsv` also provides [upper-quartile normalization](http://www.biomedcentral.com/1471-2105/11/94) factors for the bigWig coverage vectors by sample.

    4. A [BED](http://genome.ucsc.edu/FAQ/FAQformat.html#format1) file per sample per feature (introns, insertions, and deletions). The BED files are analogous to [TopHat 2](https://ccb.jhu.edu/software/tophat/index.shtml)'s BED output, in case you're used to it. Quotes below are from the [TopHat 2 manual](https://ccb.jhu.edu/software/tophat/manual.shtml): 
         * `introns_and_indels/junctions.<sample name>.bed`: "Each junction consists of two connected BED blocks, where each block is as long as the maximal overhang of any read spanning the junction. The score is the number of alignments spanning the junction."
         * `introns_and_indels/deletions.<sample name>.bed`: "chromLeft refers to the first genomic base of the deletion."
         * `introns_and_indels/insertions.<sample name>.bed`: "chromLeft refers to the last genomic base before the insertion."

    You can suppress any output type depending on your analysis goals, substantially reducing output size across many samples as well as the time Rail-RNA takes to run from end to end; see [Deliverables](deliverables.md). Indeed, since many RNA-seq analysis goals (e.g., differential expression) are achievable with just coverage information, it may not be necessary for you to ever work with a set of unwieldy BAM files.

* As of v0.2, Rail-RNA can analyze dbGaP-protected RNA-seq data in the cloud in a manner compliant with [NIH guidelines](http://www.ncbi.nlm.nih.gov/projects/gap/pdf/dbgap_2b_security_procedures.pdf). Setting an Amazon Web Services account up for analyzing protected data with Rail-RNA should less than an hour using the protocol described in [dbGaP on EMR](dbgap.md).

* Rail-RNA is designed to scale well with respect to computer cluster size and input data volume. It uses the [MapReduce](http://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf) programming model to achieve this: it divides the alignment problem up into a sequence of alternating parallelizable aggregation and computation steps, where each is performed by a different Python script operating on a different input stream. (We use [PyPy](http://pypy.org/) to improve performance.) For a single sample on single machine, it's about as fast as TopHat 2 on 8 processing cores, but it's about twice as fast as TopHat 2 on 32 cores. Rail-RNA also

    1. eliminates redundant alignment work giving its throughput (measured in, for example, number of samples analyzed per hour) better-than-linear scaling with respect to the number of samples analyzed; and

    2. borrows strength across samples in the following ways.

        * Rail-RNA filters out out likely false positive junctions that appear in only a few samples and are covered by few reads across samples. (By default, junctions that appear in <= 5% of samples and are covered by fewer than 5 reads in any one sample are filtered out.)

        * Reads in all samples are automatically realigned to transcript fragments ("isofrags") that overlap exon-exon junctions initially found in a subset of samples. This means it's likelier that exon-exon junctions covered by a small number of reads in any given sample will be discovered.

    If you want fast and furious with few processing cores, try [HISAT](https://ccb.jhu.edu/software/hisat/index.shtml), [STAR](https://github.com/alexdobin/STAR), or [Subjunc](http://subread.sourceforge.net/). If you want to perform an integrative analysis on over 20 similar RNA-seq samples---say, from the same tissue type---give Rail a shot. We're here to help if you have questions:

## Getting help

* [Read the docs](https://en.wikipedia.org/wiki/RTFM), but you're doing that right now.

* There's a [Gitter chat room](https://gitter.im/nellore/rail) where we offer live support nontrivially often. You'll need an account with GitHub to get in. Don't be scared to talk! You're encouraged to use this resource.

* You can always email [Abhi Nellore](mailto:anellore@gmail.com) with support questions. Put `[Rail-RNA support]` somewhere in the subject line, and include the version of Rail-RNA you used in your support request. To find this, enter `rail-rna --version`.

* Use the logs and error messages to figure out what went wrong yourself. Rail's source has verbose variable names, and it should be documented well enough. Since it's just some Python, you don't have to worry about compilation, either. Add try-except blocks and rerun part of your pipeline to find lines of input a script may be stumbling on. We may ask you to do things like this in the Gitter to troubleshoot, if you're willing.

## Disclaimer

Renting Amazon Web Services resources costs money, regardless of whether your run ultimately succeeds or fails. In some cases, Rail-RNA or its documentation may be partially to blame for a failed run. While we are happy to review bug reports, we do not accept responsibility for financial damage caused by these errors. Rail-RNA is provided "as is" with no warranty.