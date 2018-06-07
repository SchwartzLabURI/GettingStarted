# Preparation of NGS data for SISRS Pipeline

Getting the best output from any bioinformatics pipeline, including SISRS, requires careful upstream data preparation. For SISRS, this includes five major steps:

#### 1) Determining data availability for your study

SISRS can, in theory, use any matched set of NGS data as input (e.g. DNA-Seq, RNA-Seq, Enriched library), but it is critically important that NGS data are derived from the same type of sequencing. For example while possible, mixing DNA-Seq and RNA-Seq data will lead to the lack of assembly of non-expressed regions of the genome, limiting inference for those regions. To simplify genome assembly and maximize read-mapping equity across samples, a few basic considerations I usually use for selecting DNA-Seq data are:
- **Paired End versus Single End Reads:** SISRS currently does not use paired-end read information during analysis, so it doesn't matter if the user feeds in SE reads or PE reads. However, PE reads are useful for other downstream pipelines so if the user has the option, I would recommend getting the PE reads. 
- **Read Length/Spot Size:** 100bp reads/200bp spot length. The user should aim for similar read lengths (spot length) for all datasets. This is to try to ensure read mapping equity across what are typically small scaffolds in the composite genome.
- **Insert size:** If using PE reads, try to find paired end reads where the insert size is greater than spot length.
- **Library Selection:** Depending on the type of data you're using, this may vary. However, just be consistent and don't mix radically different library prep strategies (e.g. Random vs. Enriched)

#### 2) Quality control of NGS data to be fed into SISRS
Running FastQC both pre- and post-trimming is essential to identify data issues prior to a SISRS run. Major issues to keep an eye on include:

- **Sequence Duplication Levels:** If not using an enriched library, duplicated sequences will give an overinflated estimate of your genome coverage. Try to avoid these datasets if possible. If unavoidable, it would be best to de-duplicate reads prior to analysis. 
- **Overrepresented Sequences:** Often adapters or N's, these will likely be taken care of by basic trimming.
- **Adapter and Kmer Content:** Again, often handled by trimming. Note that if Kmer abundance is heavily weighted towards the Left or Right end of a read set, that can be specified in the trim commands (see below). 

#### 3) Adequate read trimming
I use the BBDuk software package when trimming my reads for SISRS. The basic trim command I use for DNA-Seq data is:

```
#PE Reads (Base Trim Command)
bbduk.sh maxns=0 ref=<path_to_BBUDuk>/resources/adapters.fa qtrim=w trimq=15 minlength=35 maq=25 in=<RAW_SAMPLE_LEFT>.fastq.gz in2=<RAW_SAMPLE_RIGHT>.fastq.gz out=<TRIM_SAMPLE_LEFT>.fastq.gz out2=<TRIM_SAMPLE_RIGHT>.fastq.gz

#PE Reads (End Trimming Command)
bbduk.sh maxns=0 ref=<path_to_BBUDuk>/resources/adapters.fa qtrim=w trimq=15 minlength=35 maq=25 in=<RAW_SAMPLE_LEFT>.fastq.gz in2=<RAW_SAMPLE_RIGHT>.fastq.gz out=<TRIM_SAMPLE_LEFT>.fastq.gz out2=<TRIM_SAMPLE_RIGHT>.fastq.gz mink=8 hdist=1 hdist2=0 ktrim=(r/l)

#SE Reads (Base Trim Command)
bbduk.sh maxns=0 ref=<path_to_BBUDuk>/resources/adapters.fa qtrim=w trimq=15 minlength=35 maq=25 in=<RAW_SAMPLE>.fastq.gz out=<TRIM_SAMPLE>.fastq.gz

#SE Reads (End Trimming Command)
bbduk.sh maxns=0 ref=<path_to_BBUDuk>/resources/adapters.fa qtrim=w trimq=15 minlength=35 maq=25 in=<RAW_SAMPLE>.fastq.gz out=<TRIM_SAMPLE>.fastq.gz mink=8 hdist=1 hdist2=0 ktrim=(r/l)
```

**Arguments:**
- **maxns=0**: Reads with N's filtered out. Can be adjusted based on your needs, as this is obviously the most conservative setting.
- **ref=.../resources/adapters.fa**: Sets path to BBDuk adapter database (Could be set to whatever you want to filter against)
- **qtrim=w**: Quality trim based on sliding window
- **trimq=15**: Trim to Q15 Phred bases
- **minlength=35**: Reads <35bp after trimming are discarded
- **maq=25**: Reads with an average base quality <Q25 after trimming are discarded
- **mink=8**: If trimmming Kmers from L/R end, trim Kmers 8bp or longer
- **hdist=1**: Allows a 1bp mismatch in kmer matching
- **hist2=0**: Allows 0bp mismatch when trimming shorter kmers (from mink)
- **ktrim=r/l**: Trim kmers from Right/Left end (check FastQC report)

#### 4) Subsetting based on SISRS study design
The composite genome assembly for SISRS has been validated to work well at a final genome coverage of ~10X (When all reads are combined across species, the final coverage should be ~10X the average genome size for the group of organisms). 

- For all the data that passed QC, pool data by species (Sp) to create a single fastq.gz file for each species
```
cat Sp*Left.fastq.gz > Sp_Pool_Left.fastq.gz
cat Sp*Right.fastq.gz > Sp_Pool_Right.fastq.gz
```
- For a SISRS run with X species, and an average genome size for the group of Y bp (e.g. ~3.5B for mammals), the reads from each species should be subset to (10\*Y)/X (e.g. For 50 species with an average genome size of 100Gb, subset each species to 20Gb [10*100/50])

```
# Subsetting for Mapping Dataset: If an individual species has exceptionally high coverage (e.g. >50X), consider subsetting down to 50X for more manageable file sizes
reformat.sh in=Sp_Pool_Left.fastq.gz in2=Sp_Pool_Right.fastq.gz out=Sp_Mapping_Left.fastq.gz out2=Sp_Mapping_Right.fastq.gz samplebasestarget=(50*GenomeSize)

# Subsetting for Genome Assembly: Subset each species Pooled data using BBDuk
reformat.sh in=Sp_Pool_Left.fastq.gz in2=Sp_Pool_Right.fastq.gz out=Sp_Genome_Left.fastq.gz out2=Sp_Genome_Right.fastq.gz samplebasestarget=(10*GenomeSize/NumSpecies)
```
#### 5) Composite genome assembly
Using whatever genome assembler you want, assemble the subset Genome reads into a composite genome. For example, here is a command for an assembly using Ray:
```
mpiexec -n 200 Ray -k 31 -s Sp1_A_Genome_Left.fastq.gz -s Sp1_A_Genome_Right.fastq.gz -s Sp2_A_Genome_Left.fastq.gz -s Sp2_A_Genome_Right.fastq.gz -s Sp3_A_Genome_Left.fastq.gz -s Sp3_A_Genome_Right.fastq.gz ... -o <OUTPUT_DIRECTORY>
```

## This composite genome and associated mapping reads represent the starting point for a SISRS run. 
