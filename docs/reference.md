## Command-line usage

Rail-RNA is invoked by entering
```
rail-rna <job flow> <mode> <[args]>
```

. `<job flow>` can be `prep`, `align`, or `go`; `go` executes the `prep` and `align` job flows in succession; `<mode>` can be `local`, `parallel`, or `elastic`. To get help for a given combination of job flow and mode, enter
```
rail-rna <job flow> <mode> -h
```
. Command-line arguments (args) are described in detail below.

### Required args

# `-m/--manifest <file>`

This is the path to a Rail-RNA manifest file. Its format is described in [Tutorial](tutorial#local-and-parallel-modes-drosophila-examples). Each line corresponds to a different RNA-seq sample. A line for a single-end sample looks like this---
```
<FASTQ/FASTA URL>(tab)<URL MD5 checksum or 0>(tab)<sample label>
```
---while a line for a paired-end sample looks like this---
```
<FASTQ/FASTA URL 1>(tab)<URL 1 MD5 checksum or 0>(tab)<FASTQ/FASTA URL 2>(tab)<URL 2 MD5 checksum or 0>(tab)<sample label>
```
. For non-dbGaP-protected SRA samples, a line looks like this---
```
sra:<SRA run accession number>(tab)0(tab)<sample label>
```
--while for dbGaP-protected SRA samples, a line looks like this---
```
dbgap:<SRA run accession number>(tab)0(tab)<sample label>
```
.

# `-x/--bowtie-idx/ <idx,idx | idx>`

(`local` and `parallel` modes only)

Rail-RNA needs the paths to both a Bowtie 1 index and a Bowtie 2 index of exactly the same reference FASTA. Only the basename path(s) should be specified. There are six Bowtie index files; their extensions are `.1.ebwt`, `.2.ebwt`, `.3.ebwt`, `.4.ebwt`, `.rev.1.ebwt`, and `.rev.2.ebwt`. Similarly, there are six Bowtie 2 index files; their extensions are `.1.bt2`, `.2.bt2`, `.3.bt2`, `.4.bt2`, `.rev.1.bt2`, `.rev.2.bt2`. If the Bowtie 1 and Bowtie 2 index files are in the same directory and have the same basename, only the path to this basename need be specified, like so:
```
-x <path to common Bowtie index basename>
```
. Otherwise, paths to both basenames should be specified, like so:
```
-x <path to Bowtie 1 index basename> <path to Bowtie 2 index basename>
```
. Rail-RNA is forgiving of your specifing Bowtie indexes in the reverse order.

# `-c/--core-instance-count <int>`

(`elastic` mode only)

This is the number of core instances Rail-RNA should reserve to execute the requested job flow (one of `prep`, `align`, or `go`). The instance type is by default the same as the `--master-instance-type`. Rail-RNA uses an Elastic MapReduce cluster with one master instance and `<int>` core instances.

# `-a/--assembly <choice | tgz>`

(`elastic` mode only)

This is the assembly to which Rail-RNA should align the data specified in `--manifest/-m`. Right now, only `hg19` is supported natively, but you can [build Bowtie indexes](http://bowtie-bio.sourceforge.net/manual.shtml) of an assembly, compress them into a `tar.gz`, upload the archive to S3, and then specify its full path `<tgz>`. The basename of both Bowtie indexes should be `genome`. In more detail, the archive should have the following contents.

```
index/genome.1.bt2
index/genome.2.bt2
index/genome.3.bt2
index/genome.4.bt2
index/genome.rev.1.bt2
index/genome.rev.2.bt2
index/genome.1.ebwt
index/genome.2.ebwt
index/genome.3.ebwt
index/genome.rev.1.ebwt
index/genome.rev.2.ebwt
```

Note that `index/genome.4.ebwt` is missing above. It's exactly the same as the Bowtie 2 index file `genome.4.bt2`, and it should not be included in the archive. If you've uploaded the index to, say, `s3://my-bucket/genome.tgz`, you'd specify it at the command line as
```
-a s3://my-bucket/genome.tgz
```
.

### General options

These are parameters that apply generally to Rail-RNA job flows.

# `-h/--help`

This is the go-to command-line parameter for figuring out how to use Rail-RNA without whatever you're reading now.

Boolean parameter; has no argument.

# `-v/--version`

When you [ask for help](index.md/#getting-help) from us, we'll want to know the version of Rail-RNA you've been using. Include the output of
```
rail-rna -v
```
in your support request.

Boolean parameter; has no argument.

# `-j/--json`

Rail-RNA job flows are encoded as JSON objects. A job flow's JSON is interpreted and executed by [Dooplicity's EMR simulator](https://github.com/nellore/rail/blob/master/src/dooplicity/emr_simulator.py) in `local` and `parallel` modes and Amazon Elastic MapReduce via [Dooplicity's EMR runner](https://github.com/nellore/rail/blob/master/src/dooplicity/emr_runner.py) in `elastic` mode. You can view and hack the JSON file to, say, continue a failed job flow. If you'd like to view the job flow JSON constructed by Rail-RNA without executing it, use the `-j/--json` command-line parameter.

Boolean parameter; has no argument.

# `--profile <str>`

This is the [AWS CLI profile](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-config-files) from which Rail-RNA should pull your credentials when communicating with Amazon Web Services. It's typically relevant only in `elastic` mode.

Default: credentials are pulled from the environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, but if those aren't present, the credentials are instead pulled from the default AWS CLI profile.

# `-f/--force`

By default, Rail-RNA warns you in all modes if there are existing output files present at the destination specified by `-o/--output`. Invoking `-f/--force` overwrites all output directories if they exist. This also includes all files in subdirectories of the log directory in `local` and `parallel` modes, and all files in the intermediate directory in `elastic` mode.

Unless you'll need to debug job flows, you won't miss any of these files, so feel free to force.

Boolean parameter; has no argument.

#  `--verbose`

If you're having trouble with a job flow in Rail, it may help to invoke `--verbose` to write more details to task logs. [Tutorial](tutorial.md#local-and-parallel-modes-drosophila-examples) covers an example in local mode that reviews one such log.

Boolean parameter; has no argument.

# `--do-not-bin-quals`

(`prep` and `go` job flows only)

By default, Rail-RNA places a Phred quality score from the quality sequence for each input record of a FASTQ file into one of five bins. The formula for binning mirrors [Bowtie 2's default formula](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml#bowtie2-options-mp) and thus does not compromise alignment quality. Binning quality scores does save space and speed up job flows, however, especially in `elastic` mode, where Rail-RNA always compresses intermediate data.

This option leaves quality sequences intact.

Boolean parameter; has no argument.

# `--short-read-names`

(`prep` and `go` job flows only)

SAM/BAM file outputs are sometimes considerably larger simply because read names are large. This option kills read names in favor of short yet still unique identifiers.

Boolean parameter; has no argument.

# `--skip-bad-records`

(`prep` and `go` job flows only)

By default, if an input record from a FASTA/FASTQ file is invalid, Rail-RNA's preprocessing step chokes, as [reviewed in Tutorial](tutorial.md#local-and-parallel-modes-drosophila-examples). Use this parameter to instead be robust to bad records in input files and skip them. Bad records typically occur when an input file was somehow truncated or otherwise corrupted.

Boolean parameter; has no argument.

# `--do-not-check-manifest`

(`prep` and `go` job flows only)

Rail-RNA's default behavior is to check for the existence of each input file in the manifest file before starting a job flow. This can be time-consuming for a set of input files on a remote host. If you know the files are accessible, `--do-not-check-manifest` turns the default behavior off.

Boolean parameter; has no argument.

# `--log <dir>`

(`local` and `parallel` modes only)

`<dir>` is used both for aggregating intermediate files between steps of job flows and for storing log files for diagnosing problems.

Default: "rail-rna_logs" in the current working directory

# `--intermediate <s3_dir/hdfs_dir>`

(`elastic` mode only)

This is a directory in an S3 bucket or on the temporary HDFS of an Elastic MapReduce cluster where intermediate data across all workers is aggregated. If using S3, `<s3_dir>` should begin with `s3://some-bucket`. If using HDFS, `<hdfs_dir>` should begin with `hdfs:///`. Use S3 and set `--intermediate-lifetime` to -1 to keep intermediates.

Default: argument of `-o/--output` with the string ".intermediate" tacked on

#  `-p/--num-processes <int>`

(`local` mode only)

Rail-RNA runs your job flow on up to `<int>` processes simultaneously.

Default: if you have more than one processing core available, Rail-RNA runs on as many as you have less one so it's not a resource hog. If you have only one processing core, Rail-RNA runs on it.

# `--scratch <dir>`

(`local` and `parallel` modes only)

This is a directory used for storing temporary files accumulated when aggregating intermediate data. In `parallel` mode, this should always be in some node-local directory like `/tmp`.

Default: securely created temporary directory created by Python's `tempfile.mkdtemp()`

# `--keep-intermediates`

(`local` and `parallel` modes only)

Rail-RNA's default behavior is to delete aggregated intermediate data as soon as the job flow it's running no longer needs them. Keeping them around after a job flow is complete using `--keep-intermediates` is sometimes useful for diagnosing problems, however.

Boolean parameter; has no argument.

# `-g/--gzip-intermediates`

(`local` and `parallel` modes only)

Rail-RNA has a small memory footprint but a large storage footprint. When space is tight, the storage footprint can be minimized in `local` and `parallel` modes by compressing intermediates with the `-g/--gzip-intermediates` command-line parameter. This will slow down the job flow a bit.

Boolean parameter; has no argument.

# `--gzip-level <int>`

(`local` and `parallel` modes only)

This is the level of gzip compression to use for intermediate data if it's being compressed. The extremes are 1, meaning no compression, and 9, meaning high compression. Values closer to 9 increase the time a given job flow takes significantly.

Default: 3

# `-r/--sort-memory-cap <dec>`

(`local` and `parallel` modes only)

Rail-RNA uses [GNU Coreutils](http://www.gnu.org/software/coreutils/coreutils.html) `sort` when aggregating intermediate data. `<dec>` is the maximum amount of memory (in bytes) used by `sort` per process

Default: 307200

# `--nucleotides-per-input <int>`

(`local` and `parallel` modes only)

After the `prep` job flow downloads a sample's FASTA/FASTQs from a remote server, it assigns reads to alignment tasks. Each task may be executed by a different worker. `<int>` is the maximum number of nucleotides from input reads to assign to any single task. Smaller values of `<int>` increase parallelism in the following alignment step. This command-line parameter does not apply to input data stored locally, which Rail-RNA attempts to distribute evenly among workers.

Default: 100000000

# `--do-not-gzip-input`

(`local` and `parallel` modes only)

By default, Rail-RNA gzips preprocessed input data, which is *not* considered intermediate data. Turning this option on leaves preprocessed input reads uncompressed.

# `--max-task-attempts <int>`

Sometimes, a worker attempting a task will fail for miscellaneous reasons. For example, the node on which the worker is operating may transiently run out of space. Hadoop clusters are resilient to failures like this because they automatically reattempt tasks. Rail-RNA can reattempt tasks in `local` and `parallel` modes, too, when it's not using Hadoop. `--max-task-attempts` applies in all modes: a task is reattempted up to `<int>` times by different workers in the cluster, if possible, before failing a job flow. On Elastic MapReduce, which uses Hadoop, `<int>` specifies the config parameters `mapreduce.map.maxattempts` and `mapreduce.reduce.maxattempts`.

### Algorithm options

These are parameters that tweak Rail-RNA's algorithms.

# `--bowtie2-args <str>`

These are arguments to pass to Bowtie 2, and they should be enclosed in
double-quotes. A couple arguments are always ignored, however.

1. Bowtie 2 is always run in `--local` mode by Rail-RNA, so `--end-to-end` won't be passed to it.
2. Rail-RNA bins quality scores in a particular way, and `--mp` arguments won't be passed to Bowtie 2.

Default: empty

# `-k <int>`

Tells Rail-RNA to look for up to `<int>` alignments per read per step. This takes precedence over any `-k` value specified in --bowtie2-args.

Default: no value; Rail-RNA looks for the best alignment.

#  `--isofrag-idx/-fx <file>`

This is a `tar.gz file` containing the transcript fragment (isofrag) index from previous Rail-RNA run. It is used in lieu of searching for exon-exon junctions. You'll find an `isofrags.tar.gz` file with the isofrag index in the `cross_sample_outputs/` directory of a Rail-RNA run where `idx` wasone of the [deliverables](deliverables.md), which is the default behavior.

Default: none; search for junctions.

# `--partition-length <int>`

To compute genome coverage by reads, Rail-RNA partitions the genome into bins of length `<int>` bases. Each bin is assigned to a different task to be executed by a different worker. The exon differentials described in our [paper](http://biorxiv.org/content/early/2015/05/07/019067) are summed within each bin to arrive at base-level coverage.

Partition lengths that are too large decrease parallelism and contribute to load imbalance, but partition lengths that are too small (tens or hundreds of bases) make for larger intermediate data. It's probably not necessary to change the default value.

Default: 5000

# `--max-readlet-size <int>`

Rail-RNA divides reads into short overlapping segments called readlets and subsequently aligns them to the genome to infer the presence of exon-exon junctions. For a complete description of the readletizing scheme Rail uses, see [our paper](http://biorxiv.org/content/early/2015/05/07/019067). A readlet's size does not exceed `<int>` bases.

For human-size genomes, values between 22 and 27 work well, with smaller values improving sensitivity but decreasing specificity.

Default: 25

# `--min-readlet-size <int>`

See `--max-readlet-size` for a description of readlets. Keep `<int>` above 10 to avoid making too many false positive exon-exon junction calls.

Default: 15

# `--readlet-interval <int>`

See `--max-readlet-size` for a description of readlets. Here, `<int>` is the number of bases between the start positions of consecutive overlapping readlets. Increasing this value decreases sensitivity to novel exon-exon junctions. However, we haven't found that decreasing `--readlet-interval` below 4 increaes sensitivity much.

Default: 4

#  `--readlet-config-size <int>`

After finding exon-exon junctions in samples, Rail-RNA constructs a directed acyclic graph (DAG) whose nodes represent introns. Each edge has weight equal to the number of exonic bases between the intron nodes it connects. Rail-RNA traverses the DAG to find all possible combinations of exon-exon junctions spanned by `<int>` exonic bases. It later aligns reads to transcript fragments spanning these combinations of exon-exon junctions.

We find a `--readlet-config-size` above 30 to be sufficient to resolve alignments spanning exon-exon junctions for human-size genomes.

Default: 35

# `--max-intron-size <int>`

This parameter has Rail filter out introns that span more than `<int>` bases. It significantly reduces false positive exon-exon junction calls. There are only a handful of introns that exceed 500,000 bases in the human genome, which explains the default.

Default: 500000

# `--min-intron-size <int>`

This parameter has Rail filter out introns spanning fewer than `<int>` bases. Making this too small might have Rail mistake insertions for introns.

Default: 10

# `---min-exon-size <int>`

This sets various parameters across Rail-RNA that enforce its sensitivity to exons spanning only at least `<int>` bases. Most conspicuously:

1. When searching for exon-exon junctions with readlet alignments, Rail-RNA will look for small exons that span at least `<int>` bases. Making `<int>` smaller increases false positive junction calls.
2. When traversing the DAG described under `--readlet-config-size`, Rail-RNA will eliminate junction combinations spanning exons below `<int>`.

Default: 9

# `--search-filter <choice/int>`

Rail-RNA aligns all read sequences to the genome first with Bowtie 2 in `--local` mode, which can soft-clip base sequences at read ends. Ordinarily, all read sequences with soft-clipped bases are divided into readlets and later searched for exon-exon junctions---but instead, the read sequences searched for junctions may be chosen based on how many if its bases are soft-clipped in its primary alignment instead. If `<choice/int>` is an integer `<int>`, a read sequence is only searched for junctions if >= `<int>` of its bases are soft-clipped. `<choice/int>` can also be `<choice>` from {`strict`, `mild`, `none`}. If `<choice>` is `strict`, `<int>` is set equal to `--min-exon-size`; if `<choice>` is `mild`, `<int>` is 2/3 \* `--min-exon-size`; and if `<choice>` is `none`, `<int>` is 1.

Default: none; i.e., 1

# `--junction-criteria <dec,int>`

When Rail-RNA analyzes more than one RNA-seq sample, it accumulates a master list of exon-exon junctions across samples. This master list is used to construct transcript fragments to which reads are realigned. However, when hundreds of samples are analyzed, the master list typically blows up: true positive junction calls tend to be shared by many samples, while false positive junction calls tend to be unique to small numbers of samples. As the number of samples analyzed increases, so does the master list of junctions, and false positive calls make up a larger and larger fraction of the master list. False positive calls may be due to low-quality reads or repetitive/small exonic segments on either side of an intron that make it difficult to resolve the intron's proper start and end positions in the genome.

The `--junction-criteria` parameter `<dec,int>` filters out junctions that are not either present in at least a fraction `<dec>` of samples or detected in at least `<int>` reads of one sample. This way, only junctions that are detected from a small number of reads in any one sample or from a small number of samples are eliminated, and the quality of realigned reads is improved without significantly compromising sensitivity. (See [our paper](http://biorxiv.org/content/early/2015/05/07/019067) for an elaboration of this phenomenon.) For example, specifying `--junction-criteria 0.02,7` or `--junction-criteria 0.02 7` filters out junctions that are found in fewer that 2% of samples and are detected in fewer in 7 reads in any one sample before realignment.

Default: 0.05,5

# `--normalize-percentile <dec>`

Normalization factors for coverage of the genome by primary alignments and by uniquely aligning reads are output by Rail-RNA if the `tsv` deliverable is requested; see the [appropriate section of Deliverables](deliverables.md#tsv-cross-sample-matrices). The normalization factor of a given sample can be obtained by

1. Counting the number n of bases that are covered by at least k reads, where k >= 1.
2. Writing a list of numbers where each value k occurs exactly n times.
3. Sorting the list constructed in step 2 in increasing order.
4. Selecting the `<dec>`*100th-percentile value from the list.

When `<dec>` is 0.75, this is [upper-quartile normalization](http://bib.oxfordjournals.org/content/early/2012/09/15/bib.bbs046.long).

Default: 0.75

# `--do-not-drop-polyA-tails`

By default Rail-RNA, does not map reads for which all bases to the left or right of a segment spanning `--min-exon-size` bases 

Boolean parameter; has no argument.

# `--tie-margin <int>`

Ordinarily, two alignments of a read are considered to have "tied" alignment scores if and only if the scores are exactly equal, and `<int>` is 0. However, for the purpose of identifying uniquely aligning reads and resolving primary alignments based on coverage, toggling this parameter permits a score difference of `<int>` per 100 bases among alignments whose scores are considered ties. For example, 150
and 144 are tied alignment scores for a 100-bp read when --tie-margin is 6, and if a read has alignments with both scores, it is not considered uniquely aligned.

Default: 0

# `--library-size <int>`

As described in [Deliverables](deliverables.md#bw-coverage-vectors), in the `cross_sample_outputs` directory, `[mean|median].<chr>.bw` encodes the [mean|median] coverage at each base across samples on chromosome `<chr>` by primary alignments, and `[mean|median].<chr>.bw` encodes the [mean|median] coverage at each base on chromosome `<chr>` by uniquely aligning reads. To normalize each sample's contribution to a given [mean|median], the number of reads covering a given base of the genome is multiplied by `<int>`\*1,000,000 and divided by the number of mapped reads in the sample. So `<int>` is a standard library size in millions of reads that permits easier interpretation of the mean and median coverage bigWigs.

Default: 40

### Output options

# `-o/--output <dir>`

Rail-RNA will write its output to the directory `<dir>`. In `local` mode, `<dir>` can be on either the local filesystem or on S3. In `parallel` mode, `<dir>` can be on S3 or at some path accessible to all nodes in your IPython cluster. In `elastic` mode, `<dir>` must be on S3.

Default: `./rail-rna_out` in `local` and `parallel` modes; a required parameter in `elastic` mode

# `-d/--deliverables <choice,...>`

This command-line option permits suppressing some outputs of the full Rail-RNA pipeline. Check out [Deliverables](deliverables.md) to learn more.

# `--drop-deletions`

By default, Rail-RNA counts bases deleted from the reference in a read alignment as covered by the read when computing the coverage vectors stored in bigWigs. Turning this option adjusts Rail-RNA's behavior so such deleted bases do not contribute to the coverage vectors.

Boolean parameter; has no argument.

# `--do-not-output-bam-by-chr`

By default, Rail-RNA outputs alignment BAMs by chromosome/sample to increase parallelism in a terminal output step. You can output BAMs by sample alone with this option.

Boolean parameter; has no argument.

# `--do-not-output-ave-bw-by-chr`

Obtaining a bigWig storing average (mean or median) coverage across many samples can involve much more computation than obtaining a bigWig storing coverage for a single sample. Moreover, an average coverage bigWigs can take up considerably more space than a single-sample bigWig. To mitigate the resulting load balance issues, Rail-RNA outputs average coverage bigWigs chromosome by chromosome, by default. You can output each average coverage bigWig for the entire genome instead with this option.

Boolean parameter; has no argument.

# `--indel-criteria <dec,int>`

Accumulating indels across hundreds of samples bloats the `insertions.tsv.gz` and `deletions.tsv.gz` files with many false positives for which there is little evidence. The `--indel-criteria` parameter `<dec,int>` does not write indels to these cross-sample output files unless they are either present in at least a fraction `<dec>` of samples or detected in at least `<int>` reads of one sample. `--indel-criteria`'s usage is similar to that of `--junction-criteria`.

Default: `0.05,5`

### Elastic MapReduce options

These are arguments that pertain to working with Amazon Web Services in Rail-RNA's `elastic` mode.

# `--intermediate-lifetime <int>`

It costs money to keep files around on Simple Storage Service (S3). To help save you money, Rail-RNA schedules any directory in which it stores intermediate data on S3 for deletion after `<int>` days. If `<int>` is set to `-1`, no such scheduling is done, and intermediate data is kept for indefinitely long.

Check `Lifecycle` under a given bucket's properties in the [S3 console](https://console.aws.amazon.com/s3/home) to view and adjust the rules Rail-RNA has made to delete intermediate data.

Default: 4

# `--name <str>`

It can be useful to assign a distinctive name `<str>` to an Elastic MapReduce job flow if you're working with many in the [console](https://console.aws.amazon.com/elasticmapreduce).

Default: `Rail-RNA Job Flow`

# `--log-uri <s3_dir>`

This is the directory on S3 to which Elastic MapReduce will ultimately copy Hadoop logs.

Default: argument of `-o/--output` with the string ".logs" tacked on

# `--ami-version <str>`

This is the version of the Amazon Machine Image to load onto each node of the Elastic MapReduce cluster. Changing `<str>` is not recommended

Default: `3.8.0`

# `--visible-to-all-users`

This makes the Elastic MapReduce cluster accessible to all IAM users associated with the main Amazon Web Services account.

Boolean parameter; has no argument.

# `--action-on-failure <choice>`

This is the action Elastic MapReduce should take if the job flow it's running fails on a given step. `<choice>` is one of {`TERMINATE_JOB_FLOW`, `CANCEL_AND_WAIT`, `CONTINUE`, `TERMINATE_CLUSTER`}.

Default: `TERMINATE_JOB_FLOW`

# `--master-instance-count <int>`

This is the number of master instances to include in your Elastic MapReduce cluster. It's only useful to include more than one in case a master instance dies; in general, `<int>` shouldn't be changed.

Default: 1

# `--task-instance-count <int>`

Task instances are nodes in an Elastic MapReduce cluster that do not store data. Your job flow will continue even if they're lost, which can happen if nodes go down or--more likely--they're [spot instances](http://aws.amazon.com/ec2/purchasing-options/spot-instances/) whose maximum bid price no longer exceeds the market prices. You should consider using task instances only if you'll be bidding for them on the spot market at the rate `--task-instance-bid-price`.

Default: 0

# `--master-instance-bid-price <dec>`

If you'd like to use the [spot market](http://aws.amazon.com/ec2/purchasing-options/spot-instances/) to potentially save money on your job flow, you can set the bid price (in dollars/hour) for the master instance group here. Invoke this command-line parameter only if master instances should be spot.

Default: none; use on-demand master instances

# `--core-instance-bid-price <dec>`

If you'd like to use the [spot market](http://aws.amazon.com/ec2/purchasing-options/spot-instances/) to potentially save money on your job flow, you can set the bid price (in dollars/hour) for the core instance group here. Invoke this command-line parameter only if core instances should be spot.

Default: none; use on-demand core instances

# `--task-instance-bid-price <dec>`

If you'd like to use the [spot market](http://aws.amazon.com/ec2/purchasing-options/spot-instances/) to potentially save money on your job flow, you can set the bid price (in dollars/hour) for the task instance group here (if you have any task instances). Invoke this command-line parameter only if you are using task instances. Indeed, there is typically no point to task instances unless they're spot instances: you may as well use on-demand core instances and be afforded extra space.

# `--master-instance-type <choice>`

This is the [instance type](http://aws.amazon.com/ec2/instance-types/) of the master instance group of your Elastic MapReduce cluster.

Default: `c3.2xlarge`

# `--core-instance-type <choice>`

This is the [instance type](http://aws.amazon.com/ec2/instance-types/) of the core instance group of your Elastic MapReduce cluster.

Default: argument of `--master-instance-type`

# `--task-instance-type <choice>`

This is the [instance type](http://aws.amazon.com/ec2/instance-types/) of the core instance group of your Elastic MapReduce cluster.

Default: argument of `--master-instance-type` if there are any task instances

# `--ec2-key-name <str>`

You can specify an Elastic Compute Cloud (EC2) key pair name `<str>` for SSHing to the master instance of your Elastic MapReduce cluster while your job flow is running.

Default: unspecified, which means you can't SSH to the master instance of your cluster

# `--keep-alive`

This option keeps an Elastic Mapreduce cluster alive when all its assigned steps are complete. Use `--keep-alive` in conjunction with `--termination-protected` and `--ec2-key-name` to be able to SSH to an Elastic MapReduce cluster and diagnose problems. However, you should make sure to terminate the cluster manually when you're finished.

Boolean parameter; has no argument.

# `--termination-protected`

This option protects an Elastic MapReduce cluster from termination in case of step failure. Use `--termination-protected` in conjunction with `--keep-alive` and `--ec2-key-name` to be able to SSH an Elastic MapReduce cluster and diagnose problems. However, you should make sure to terminate the cluster manually when you're finished.

# `--region <choice>`

This specifies the region in which your job flow will be run. Valid regions are given [here](http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/emr-plan-region.html).

Default: the region from your `--profile`, but if that's unavailable, `us-east-1` (US Standard)

# `--service-role <str>`

You should have set up your IAM service role by entering `aws emr create-default-roles` after installing the AWS CLI. If for some reason related to permissioning you need to use a different IAM service role, specify its name as `<str>`.

Default: taken from `--profile` if available; otherwise, attempts `EMR_DefaultRole`

# `--instance-profile <str>`

You should have set up your IAM EC2 instance profile by entering `aws emr create-default-roles` after installing the AWS CLI. If for some reason related to permissioning you need to use a different IAM EC2 instance profile, specify its name as `<str>`.

Default: taken from `--profile` if available; otherwise, attempts `EMR_EC2_DefaultRole`
