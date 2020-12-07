# RNA_seq-STAR
###Download STAR###
Obtain STAR source from https://github.com/alexdobin/STAR


Add the following to your .bashrc file and source it:
`export PATH=/path/to/STAR/bin/:$PATH`



###Generate Reference Genome
Before using STAR, a reference genome must be built using STAR's `genomeGenerate` mode. This requires a genome fasta file and GTF/GFF reference annotation. This can be achieved with the following command:

```
STAR --runThreadN {number of cores} --runMode genomeGenerate --genomeDir /path/to/resulting/STAR/genome/ --genomeFastaFiles /path/to/genome/fasta/file --sjdbGTFfile /path/to/GTF/or/GFF --sjdbOverhang {read length - 1}
```

Note: the `--sjdbOverhang` is dependent on your read length of your fastq files. 100 is the default value and said to work well in most cases.


### STAR Alignment ###
After you have built a genome for STAR, you can proceed to align single-end or paired-end fastq files to this reference using the following command:

```
STAR --runMode alignReads --outSAMtype BAM Unsorted --readFilesCommand zcat --genomeDir /path/to/STAR/genome/folder --outFileNamePrefix {sample name}  --readFilesIn  /path/to/R1 /path/to/R2
```

Note:
-` --outSAMtype` can be left as `BAM Unsorted` if you are going to utilize HTSeq for read counting, since HTSeq requires .bam files to be name sorted (which you can easily pipe samtools prior) or you may use the option `BAM SortedByCoordinate` if you are aligning reads to generate tdfs for viewing.
- `--readFilesCommand` should remain `zcat` if raw samples are gzipped (i.e. `.fastq.gz` extension). Omit this flag otherwise.
- R1 and R2 can accommodate comma separted input which enables mapping of technical replicates, namely fastq's for the same sample sequenced on multiple lanes. Just make sure R2 technical replicates are in the same order as R1.

Below is a concise example of how to loop through an entire directory. Since STAR is incredibly quick, it's suitable to run each alignment in serial:

```
for i in *_R1.fastq.gz; do
STAR --runMode alignReads --genomeLoad  LoadAndKeep --readFilesCommand zcat --outSAMtype BAM Unsorted --genomeDir /path/to/STAR/genome --readFilesIn $i ${i%_R1.fastq.gz}_R2.fastq.gz --runThreadN 10 --outFileNamePrefix ${i%_R1.fastq.gz}
done
```


Note: The  `--genomeLoad  LoadAndKeep` option will save the built genome into memory allowing for faster alignment
