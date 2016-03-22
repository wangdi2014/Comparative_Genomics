# Bacterial Comparative Genomics Workshop

- [Day 1 Morning](https://github.com/alipirani88/Comparative_Genomics#day-1-morning)
***
	- [Getting your data onto Flux and setting up Environment variable](https://github.com/alipirani88/Comparative_Genomics#getting-your-data-onto-glux-and-setting-up-environment-variable)
	- [Quality Control using FastQC](https://github.com/alipirani88/Comparative_Genomics#quality-control-using-fastqc)
	- [Quality Trimming using Trimmomatic](https://github.com/alipirani88/Comparative_Genomics#quality-trimming-using-trimmomatic)

- [Day 1 Afternoon](https://github.com/alipirani88/Comparative_Genomics#day-1-afternoon)
***
	- [Read Mapping](https://github.com/alipirani88/Comparative_Genomics#read-mapping)
	- [Variant Calling](https://github.com/alipirani88/Comparative_Genomics#variant-calling-and-filteration)
	- [Visualize BAM/VCF files in IGV/ACT](https://github.com/alipirani88/Comparative_Genomics#visualize-bam-and-vcf-files-in-igv-or-act)

- [Day 2 Morning](https://github.com/alipirani88/Comparative_Genomics#day-2-morning)
***
	- [Genome Assembly](https://github.com/alipirani88/Comparative_Genomics#genome-assembly)
	- [Assembly evaluation](https://github.com/alipirani88/Comparative_Genomics#assembly-evaluation)
	- [Compare assembly to reference genome and Post-assembly genome improvement](https://github.com/alipirani88/Comparative_Genomics#compare-assembly-to-reference-genome-and-post-assembly-genome-improvement)
	- [Map reads to the final ordered assembly](https://github.com/alipirani88/Comparative_Genomics#map-reads-to-the-final-ordered-assembly)
	- [Genome Annotation](https://github.com/alipirani88/Comparative_Genomics#genome-annotation)
	- [Visualize multiple assemblies](https://github.com/alipirani88/Comparative_Genomics#visualize-multiple-assemblies)

# Day 1 Morning
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

## Getting your data onto Flux and setting up Environment variable

**Log in to Flux**

```
ssh user@flux-login.engin.umich.edu
```

> *user = your umich unique name

**Set up your .bashrc file so your environment is all set for genomic analysis!**

Environment variables are a way of passing information from the shell to programs when you run them. Programs look "in the environment" for particular variables and if they are found will use the values stored. Some are set by the system, others by you, yet others by the shell, or any program that loads another program. All the softwares/tools that we need in this workshop are installed in a directory "/scratch/micro612w16_fluxod/shared/bin/". We will set the environment variable PATH in .bashrc file by exporting the required paths.

>i. Make a backup copy of ~/.bashrc file in case something goes wrong. 
	
```
cp ~/.bashrc ./bashrc_backup
```
	
>ii. Add a line to your .bashrc file that points to required Perl library directories.

```
export PERL5LIB=/scratch/micro612w16_fluxod/shared/bin/PAGIT/lib:/scratch/micro612w16_fluxod/shared/bin/vcftools_0.1.12b/perl:$PERL5LIB
```

>iii. Add entries in your .bashrc file to add required genomics programs to your path variable.

```
export PATH=$PATH: /scratch/micro612w16_fluxod/shared/bin/mauve_snapshot_2015-02-13/linux-x64/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/mauve_snapshot_2015-02-13/linux-x64/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/blast/bin/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/vcftools_0.1.12b/perl/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/tabix-0.2.6/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/bwa-0.7.12/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/Trimmomatic/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/bcftools-1.2/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/samtools-1.2/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/sratoolkit/bin/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/Spades/bin/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/FastQC/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/GenomeAnalysisTK-3.3-0/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/picard-tools-1.130/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/qualimap_v2.1/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/vcftools_0.1.12b/bin/
export PATH=$PATH:/scratch/micro612w16_fluxod/shared/bin/sratoolkit/bin/
```

>iv. Source your .bashrc file

```
source .bashrc
```

## Quality Control using [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/ "FastQC homepage")
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

>i. Execute the following commands to copy files for this morning’s exercises to your scratch directory: 

```
cd /scratch/micro612w16_fluxod/username
cp -r /scratch/micro612w16_fluxod/shared/data/day1_morn/ ./
cd /scratch/micro612w16_fluxod/username/day1_morn/
ls
```

As soon as you receive your sample data from sequencing centre, the first thing you do is check its quality using quality control tool such as FastQC. But before carrying out extensive QC, you can run a bash one-liner to get some basic statistics about the raw reads.

Run the following command to print total number of reads in each file, total number of unique reads, percentage of unique reads, most abundant sequence(useful to find adapter sequences or contamination), its frequency, and frequency of that sequence as a proportion of the total reads.

```
for i in *.gz; do zcat $i | awk '((NR-2)%4==0){read=$1;total++;count[read]++}END{for(read in count){if(!max||count[read]>max) {max=count[read];maxRead=read};if(count[read]==1){unique++}};print total,unique,unique*100/total,maxRead,count[maxRead],count[maxRead]*100/total}'; done
```

You can find more of such bash one-liners at Stephen Turner's github [page.](https://github.com/stephenturner/oneliners)

Now we will run FastQC on these raw data to assess its quality. FastQC is a quality control tool that reads in sequence data in a variety of formats(fastq, bam, sam) and can either provide an interactive application to review the results or create an HTML based report which can be integrated into any pipeline. It is generally the first step that you take upon receiving the sequence data from sequencing facility to get a quick sense of its quality or whether it exhibits any unusual properties(contamination or interesting biological features)

>ii. In your day1_morn directory, create a new directory for saving FastQC results.

```
mkdir Rush_KPC_266_FastQC_results
mkdir Rush_KPC_266_FastQC_results/before_trimmomatic
```

>iii. Verify that FastQC is in your path by invoking it from command line.

```
fastqc -h
```
> FastQC can be run in two modes: "command line" or as a GUI (graphical user interface). We will be using command line version of it.

>iv. Get an interactive cluster node to start running programs

>v. Run FastQC to generate quality report of sequence reads.

```
fastqc -o Rush_KPC_266_FastQC_results/before_trimmomatic/ Rush_KPC_266_1_combine.fastq.gz Rush_KPC_266_2_combine.fastq.gz --extract
```

This will generate the results directory for forward and reverse fastq reads called Rush_KPC_266_1_combine_fastqc and Rush_KPC_266_2_combine_fastqc in output folder provided with -o argument. The summary.txt file in these directories indicates if the data passed different quality control tests. You can visualize and assess the quality of data by opening html report in a local browser.


>vi. Exit your cluster node so you don’t waste cluster resources and $$$!

>vii. Download FastQC report to your home computer to examine

```
scp username@flux-xfer.engin.umich.edu:/scratch/micro612w16_fluxod/username/day1_morn/Rush_KPC_266_FastQC_results/before_trimmomatic/*.html /path-to-local-directory/
```

The analysis in FastQC is performed by a series of analysis modules. The left hand side of the main interactive display or the top of the HTML report show a summary of the modules which were run, and a quick evaluation of whether the results of the module seem entirely normal (green tick), slightly abnormal (orange triangle) or very unusual (red cross). 

`Screenshots explanation.`
`Explaining Summary results, Basic statistics, per base sequence quality, overrepresented sequences(adapters) from before trimmomatic report.`

Notice the quality drop(per base sequence quality graph) at the end of Rush_KPC_266_2_combine_fastqc.html report. This is commonly observed in illumina samples that as the number of sequencing cycles performed is increased the average quality of the base calls, as reported by the Phred Scores produced by the sequencer falls. Check the overrepresented sequences graph and the kind of adapters that were used for sequencing these samples.

Check out [this](https://sequencing.qcfail.com/articles/loss-of-base-call-accuracy-with-increasing-sequencing-cycles/) detailed explaination as to why quality drops with increasing sequencing cycles.

> [A video FastQC walkthrough created by FastQC developers](https://www.youtube.com/watch?v=bz93ReOv87Y "FastQC video") 

## Quality Trimming using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic "Trimmomatic Homepage")
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

Now we will run Trimmomatic on these raw data to remove low quality reads and adapters and run FastQC again to check if the quality improved after running trimmomatic. 

>i. Get an interactive cluster node to start running programs


>ii. Create these output directories in your day1_morn folder to save trimmomatic results

```
mkdir Rush_KPC_266_trimmomatic_results
mkdir Rush_KPC_266_trimmomatic_results_with_headcrop/
mkdir Rush_KPC_266_FastQC_results/after_trimmomatic_headcrop/
```

>iii. Load latest version of java and try to invoke trimmomatic from command line.

```
module load lsa java/1.8.0

java -jar /scratch/micro612w16_fluxod/shared/bin/Trimmomatic/trimmomatic-0.33.jar –h
```

explaining parameters and its default value. Adapter file. Changing only SLIDINGWINDOW parameter from default 4:15 to 4:20 for raw reads.

>iv. Run the below trimmomatic commands on raw reads(explaining parameters and its default value. Adapter file.)

```
time java -jar /scratch/micro612w16_fluxod/shared/bin/Trimmomatic/trimmomatic-0.33.jar PE Rush_KPC_266_1_combine.fastq.gz Rush_KPC_266_2_combine.fastq.gz Rush_KPC_266_trimmomatic_results/forward_paired.fq.gz Rush_KPC_266_trimmomatic_results/forward_unpaired.fq.gz Rush_KPC_266_trimmomatic_results/reverse_paired.fq.gz Rush_KPC_266_trimmomatic_results/reverse_unpaired.fq.gz ILLUMINACLIP:/scratch/micro612w16_fluxod/shared/bin/Trimmomatic/adapters/TruSeq3-PE.fa:2:30:10:8:true SLIDINGWINDOW:4:20 MINLEN:40 HEADCROP:0
```

>v. Now create new directories in day1_morn folder and Run FastQC on these trimmomatic results.

```
mkdir Rush_KPC_266_FastQC_results/after_trimmomatic

fastqc -o Rush_KPC_266_FastQC_results/after_trimmomatic/ Rush_KPC_266_trimmomatic_results/forward_paired.fq.gz Rush_KPC_266_trimmomatic_results/reverse_paired.fq.gz --extract
```

`screenshots explaination`
`before trimmomatic and after trimmomatic explanation: summary, quality, overrepresented sequences`

>-- How head bases in per base sequence content graph are imbalanced? The cross signal sign for that graph? How you can fix it by using headcrop parameter in trimmomatic? 

>vi. Run trimmomatic with headcrop 9

```
time java -jar /scratch/micro612w16_fluxod/shared/bin/Trimmomatic/trimmomatic-0.33.jar PE Rush_KPC_266_1_combine.fastq.gz Rush_KPC_266_2_combine.fastq.gz Rush_KPC_266_trimmomatic_results_with_headcrop/forward_paired.fq.gz Rush_KPC_266_trimmomatic_results_with_headcrop/forward_unpaired.fq.gz Rush_KPC_266_trimmomatic_results_with_headcrop/reverse_paired.fq.gz Rush_KPC_266_trimmomatic_results_with_headcrop/reverse_unpaired.fq.gz ILLUMINACLIP:/scratch/micro612w16_fluxod/shared/bin/Trimmomatic/adapters/TruSeq3-PE.fa:2:30:10:8:true SLIDINGWINDOW:4:20 MINLEN:40 HEADCROP:9
```
		
>-- explain per base sequence content changed to just warning from cross sign.
`screenshot explanation`
`After trimmomatic headcrop screenshot explanation of summary and per base sequence content`

>vii. Run FastQC on updated trimmomatic results and check report on your local computer

```
fastqc -o Rush_KPC_266_FastQC_results/after_trimmomatic_headcrop/ --extract -f fastq Rush_KPC_266_trimmomatic_results_with_headcrop/forward_paired.fq.gz Rush_KPC_266_trimmomatic_results_with_headcrop/reverse_paired.fq.gz
```

[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

# Day 1 Afternoon

## Read Mapping
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

**1. Create a directory to save results and run trimmomatic**

```
 mkdir Rush_KPC_266_varcall_result
```

```
 java -jar /scratch/micro612w16_fluxod/shared/bin/Trimmomatic/trimmomatic-0.33.jar PE Rush_KPC_266_1_combine.fastq.gz
Rush_KPC_266_2_combine.fastq.gz Rush_KPC_266_varcall_result/forward_paired.fq.gz Rush_KPC_266_varcall_result/forward_unpaired.fq.gz Rush_KPC_266_varcall_result/reverse_paired.fq.gz Rush_KPC_266_varcall_result/reverse_unpaired.fq.gz ILLUMINACLIP:/scratch/micro612w16_fluxod/shared/bin/Trimmomatic/adapters/TruSeq3-PE.fa:2:30:10:8:true SLIDINGWINDOW:4:20 MINLEN:40
HEADCROP:0
```

**2. Map your reads against a finished reference genome using [BWA](http://bio-bwa.sourceforge.net/bwa.shtml "BWA manual")**

>i. Create BWA index from Reference fasta file:

```
bwa index /scratch/micro612w16_fluxod/shared/bin/reference/KPNIH1/KPNIH1.fasta
```

>index file usage 

>ii. Align reads to reference and output into SAM file

```
bwa mem -M -R "@RG\tID:96\tSM:Rush_KPC_266_1_combine.fastq.gz\tLB:1\tPL:Illumina" -t 8
/scratch/micro612w16_fluxod/shared/bin/reference/KPNIH1/KPNIH1.fasta Rush_KPC_266_varcall_result/forward_paired.fq.gz Rush_KPC_266_varcall_result/reverse_paired.fq.gz > Rush_KPC_266_varcall_result/Rush_KPC_266__aln.sam
```

> -R readgroup parameter; what does it say?

**3. SAM/BAM manipulation and variant calling using [Samtools](http://www.htslib.org/doc/samtools.html "Samtools Manual")**

>i. Change directory to results folder:

```
cd Rush_KPC_266_varcall_result
```

>ii. Convert SAM to BAM using SAMTOOLS:

```
samtools view -Sb Rush_KPC_266__aln.sam > Rush_KPC_266__aln.bam
```

>iii. Sort BAM file using SAMTOOLS:

```
samtools sort Rush_KPC_266__aln.bam Rush_KPC_266__aln_sort
```

**4. Mark duplicates(PCR optical duplicates) and remove them using [PICARD](http://broadinstitute.github.io/picard/command-line-overview.html#MarkDuplicates "Picard MarkDuplicates")**

>i. Create a dictionary for reference fasta file required by PICARD(If KPNIH1.dict doesn’t exist).
 
```
java -jar /scratch/micro612w16_fluxod/shared/bin/picard-tools-1.130/picard.jar CreateSequenceDictionary REFERENCE=/scratch/micro612w16_fluxod/shared/bin/reference/KPNIH1/KPNIH1.fasta OUTPUT=/scratch/micro612w16_fluxod/shared/bin/reference/KPNIH1/KPNIH1.dict
```

>ii. Run PICARD for removing duplicates.

```
java -jar /scratch/micro612w16_fluxod/shared/bin/picard-tools-1.130/picard.jar MarkDuplicates REMOVE_DUPLICATES=true INPUT=Rush_KPC_266__aln_sort.bam OUTPUT= Rush_KPC_266__aln_marked.bam METRICS_FILE=Rush_KPC_266__markduplicates_metrics
CREATE_INDEX=true VALIDATION_STRINGENCY=LENIENT
```

>PCR optical duplicates; why remove that?

>iii. Sort these marked BAM file again (for downstream compatibility)

```
samtools sort Rush_KPC_266__aln_marked.bam Rush_KPC_266__aln_sort
```

>iv. Index these marked bam file using SAMTOOLS

```
samtools index Rush_KPC_266__aln_sort.bam
```

## Variant Calling and Filteration
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

**1. Call variants using [samtools](http://www.htslib.org/doc/samtools.html "samtools manual") mpileup and [bcftools](https://samtools.github.io/bcftools/bcftools.html "bcftools")**

```
samtools mpileup -ug -f /scratch/micro612w16_fluxod/shared/bin/reference/KPNIH1/KPNIH1.fasta Rush_KPC_266__aln_sort.bam | bcftools call -O v -v -c -o Rush_KPC_266__aln_mpileup_raw.vcf
```

>**-g generate genotype likelihood in bcf format   
>**mpileup format   
>**-c samtools consensus caller

**2. Variant filtering and processed file generation using GATK and vcftools**

>i. Variant filtering using [GATK](https://www.broadinstitute.org/gatk/guide/tooldocs/org_broadinstitute_gatk_tools_walkers_filters_VariantFiltration.php "GATK Variant Filteration"):

```
java -jar /scratch/micro612w16_fluxod/shared/bin/GenomeAnalysisTK-3.3-0/GenomeAnalysisTK.jar -T VariantFiltration -R
/home2/apirani/bin/reference/KPNIH1/KPNIH1.fasta -o Rush_KPC_266__filter_gatk.vcf --variant Rush_KPC_266__aln_mpileup_raw.vcf --filterExpression "FQ < 0.025 && MQ > 50 && QUAL > 100 && DP > 15" --filterName pass_filter
```

>**FQ, MQ, DP, QUAL
> [More Info on VCF format specifications](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwjzkcSP4MfLAhUDyYMKHU3yDwMQFggjMAA&url=https%3A%2F%2Fsamtools.github.io%2Fhts-specs%2FVCFv4.2.pdf&usg=AFQjCNGFka33WgRmvOfOfp4nSaCzkV95HA&sig2=6Xb3XDaZfghadZfcnnPQxw&cad=rja "VCF format Specs.")

>ii. Remove indels and keep only variants that passed filter parameter from VCF file using [vcftools](http://vcftools.sourceforge.net/man_latest.html vcftools manual):
 
 
```
vcftools --vcf Rush_KPC_266__filter_gatk.vcf --keep-filtered pass_filter --remove-indels --recode --recode-INFO-all --out
Rush_KPC_266__filter_onlysnp
```

>>**why remove indels  

>iii. Generate Consensus fasta file from filtered variants using vcftools:

```
bgzip Rush_KPC_266__filter_onlysnp.recode.vcf
tabix Rush_KPC_266__filter_onlysnp.recode.vcf.gz
```

>iv. Generate Statistics report using samtools, vcftools and qualimap

```
commands here
```

> open statistics file and see details
> open qualimap pdf report in your local system
> depth of coverage and other details

## Visualize BAM and VCF files in IGV or ACT
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

> Required Input files: KPNIH1 reference fasta and genbank file, Rush_KPC_266__aln_marked.bam and Rush_KPC_266__aln_marked.bai, Rush_KPC_266__aln_mpileup_raw.vcf and Rush_KPC_266__filter_onlysnp.recode.vcf

```
screenshots explanation here
```

[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)


# Day 2 Morning
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)

## Genome Assembly

## Assembly evaluation

## Compare assembly to reference genome and Post-assembly genome improvement

## Map reads to the final ordered assembly

## Genome Annotation

## Compare multiple assemblies






