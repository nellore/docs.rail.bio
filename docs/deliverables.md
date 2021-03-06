## Choosing deliverables

There are several classes of outputs. Each is described in a different section below. You can toggle which outputs are written with the `-d/--deliverables` command-line parameter for `go` and `align` job flows. By default `--deliverables` is set to `tsv idx bw bed`, which is all the outputs described below besides the alignment BAMs. (Do you really need them? Suppressing the output saves a lot of space--which is especially useful on S3, where storage is rented.) If you do need BAM output, invoke `--deliverables tsv idx bw bed bam`.

### `tsv`: cross-sample tables

These are gzip-compressed cross-sample outputs. They appear in the `cross_sample_results` subdirectory of the output directory.

  * `counts.tsv.gz`: A labeled matrix whose (i, j)th element takes the form "x<sub>ij</sub>,y<sub>ij</sub>", where x<sub>ij</sub> is the number of primary alignments in sample i to group j, and y<sub>ij</sub> is the number of uniquely aligning reads in group j. Here, a group is a chromosome, the set of unmapped reads U, the set of mapped reads M, or the set of all reads A; and "uniquely aligning read" means that there is exactly one alignment of the read with the highest alignment score. For group U, x<sub>ij</sub> = y<sub>ij</sub>. For group M, x<sub>ij</sub> is the sum of all the x<sub>ij</sub>s in the chromosome columns. For group A, x<sub>ij</sub> is the total number of reads, while y<sub>ij</sub> is the sum of the y<sub>ij</sub>s in groups U and M.
  * `junctions.tsv.gz`: a labeled matrix whose (i, j)th element is the number of reads in sample j covering intron i. Introns are in the first column. Each one takes the form

        <chromosome>;<strand (+/-)>;<1-based start position (inclusive)>;<1-based end position (exclusive)>.

    Sample names are in the first row.
  * `[insertion|deletion]s.tsv.gz`: a labeled matrix whose (i, j)th element is the number of reads in sample j covering [insertion|deletion] i. [Insertion|deletion]s are in the first column. An insertion takes the form

        <chromosome>;<inserted base sequence>;<1-based position of the last base before the insertion>;<1-based position of the last base before the insertion>

    A deletion takes the form
  
        <chromosome>;<deleted base sequence>;<1-based position of the first deleted base>;<1-based position of the first base after the deletion>

  * `normalization.tsv.gz`: each row takes the form `<sample name>`(tab)<`--normalize-percentile` normalization factor for coverage vector composed of primary alignments>(tab)<`--normalize-percentile` normalization factor for coverage vector composed of unique alignments>. The default percentile is `0.75`, corresponding to [upper-quartile normalization](http://www.biomedcentral.com/1471-2105/11/94).

### `idx`: isofrag index

This refers to the file `isofrags.tar.gz` that appears in the `cross_sample_results` subdirectory of the output directory. In general, it should be left gzipped; it can be used to align a new sample to the transcript fragments obtained from an old Rail-RNA run. Say you've aligned 500 samples from the same tissue type with Rail, and you think the list of exon-exon junctions you found is pretty comprehensive. Now say you just obtained another 10 samples from the same tissue type, and you want to align them. You can skip looking for novel junctions in those 10 samples by passing Rail-RNA's `go` or `align` job flow the path to this file as an argument of the `--isofrag-idx` command-line parameter.

### `bw`: coverage vectors

These are coverage bigWigs, and they appear in the `coverage_bigwigs` subdirectory of the output directory. There are two for each sample. One is called `<sample name>.bw`, and it encodes coverage of the genome by primary alignments. The other is called `<sample name>.unique.bw`, and it encodes coverage of the genome by uniquely aligning reads, where a uniquely aligning read has exactly one alignment with the highest alignment score. There are also coverage bigwigs for each chromosome. `[mean|median].<chr>.bw` encodes the [mean|median] coverage at each base across samples on chromosome `<chr>` by primary alignments, and `[mean|median].<chr>.bw` encodes the [mean|median] coverage at each base on chromosome `<chr>` by uniquely aligning reads. To normalize each sample's contribution to the [mean|median], the number of reads covering a given base of the genome is multiplied by some library size parameter (by default 40 million) and divided by the number of mapped reads in the sample. Leo Collado-Torres describes how to use bigWigs as input to downstream analysis tools like [`derfinder`](http://bioconductor.org/packages/release/bioc/html/derfinder.html), [`edgeR-robust`](http://bioconductor.org/packages/release/bioc/html/edgeR.html), and [`DESeq2`](http://bioconductor.org/packages/release/bioc/html/DESeq.html) [here](http://lcolladotor.github.io/protocols/bigwig_DEanalysis/). Note that a bigWig is typically an order of magnitude smaller than a BAM.

### `bam`: alignments

These are alignment BAMs and their corresponding indexes (BAIs) that appear in the `alignments` subdirectory of the output directory. By default, there's one BAM per sample per chromosome taking the form `alignments.<sample name>.<chr>.bam`. You can use the `--do-not-output-bam-by-chr` command-line parameter for `go` and `align` job flows to output one BAM per sample. In that case, the filename takes the form `alignments.<sample name>.bam`. Note that outputting BAMs by chromosome is in general faster because it increases parallelism.

### `bed`: junctions and indels

These are BED files that mirror TopHat 2's output BEDs. They appear in the `junctions_and_indels` subdirectory of the output directory. For each sample `<sample name>`, there are three BEDs: `junctions.<sample name>.bed`, `insertions.<sample name>.bed`, and `deletions.<sample name>.bed`. Quotes in the following statements are from the [TopHat 2 manual](https://ccb.jhu.edu/software/tophat/manual.shtml). Recall that BED always uses 0-based coordinates.
  * `junctions.<sample name>.bed`: "Each junction consists of two connected BED blocks, where each block is as long as the maximal overhang of any read spanning the junction. The score is the number of alignments spanning the junction."
  * `deletions.<sample name>.bed`: "chromLeft refers to the first genomic base of the deletion."
  * `insertions.<sample name>.bed`: "chromLeft refers to the last genomic base before the insertion."