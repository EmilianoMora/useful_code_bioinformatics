# Structural variants around *P. boveana* S-locus
following [analysis of the S-locus of P. edelbergii](https://gitlab.com/jackpotente/pedelbergii/-/blob/main/structural_variants.md?ref_type=heads#coverage-and-heterozygosity-of-the-s-locus)

[[_TOC_]]

## Trimming illumina reads
$ cd /home/ubuntu/barbara/analysis/trimmed_IlluminaReads/genomePlant/trimmomatic.sh
```
inFolder=/home/ubuntu/my_data-2/raw_data/boveana/WGS_shotgun/NovaSeq_20220128_NOV1132_o26645_DataDelivery
outFolder=/home/ubuntu/barbara/analysis/trimmed_IlluminaReads/genomePlant

for i in `cat sample_prefixes.txt`;
do
java -jar /home/ubuntu/giacomo-2/programs/Trimmomatic-0.38/trimmomatic-0.38.jar \
PE -phred33 \
$inFolder/$i\_R1.fastq.gz \
$inFolder/$i\_R2.fastq.gz \
$outFolder/$i\.qc.R1.paired.fastq \
$outFolder/$i\.qc.R1.unpaired.fastq \
$outFolder/$i\.qc.R2.paired.fastq \
$outFolder/$i\.qc.R2.unpaired.fastq \
ILLUMINACLIP:/home/ubuntu/giacomo-2/programs/Trimmomatic-0.38/adapters/adapters-illumina.fa:2:40:15 \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
done
```

## Mapping trimmed illumina reads on boveana assembly using bwa, remove duplicates and create VCF from aligning the Illumina reads vs the assembly
$ cd /home/ubuntu/barbara/analysis/vcf/boveana/run_bwa_to_vcf.sh
```
PROGRAMS=/home/ubuntu/my_data/Narcis/executables
VCFTOOLS_PATH=/home/ubuntu/exe_primrose/vcftools/src/cpp
BGZIP_PATH=/home/ubuntu/giacomo-2/programs/htslib/
FASTQ1=20220128.A-Pboveana_R1.fastq.gz
NAME1=$(basename $FASTQ1 _R1.fastq.gz)
INDEX=/home/ubuntu/barbara/analysis/HiC/YaHS/Pboveana_polca/Reorganization_FINAL_FASTA2/pbov_polca.filt.fasta
REFERENCE=/home/ubuntu/barbara/analysis/HiC/YaHS/Pboveana_polca/Reorganization_FINAL_FASTA2/pbov_polca.filt.fasta

###################################################

# trimming
#mkdir trimmed_reads

#java -jar /home/ubuntu/giacomo-2/programs/Trimmomatic-0.38/trimmomatic-0.38.jar \
#PE -phred33 \
#$NAME1\_R1.fastq.gz \
#$NAME1\_R2.fastq.gz \
#trimmed_reads/$NAME1\.qc.R1.paired.fastq \
#trimmed_reads/$NAME1\.qc.R1.unpaired.fastq \
#trimmed_reads/$NAME1\.qc.R2.paired.fastq \
#trimmed_reads/$NAME1\.qc.R2.unpaired.fastq \
#ILLUMINACLIP:/home/ubuntu/giacomo-2/programs/Trimmomatic-0.38/adapters/adapters-illumina.fa:2:40:15 \
#LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36


# Alignment of short reads + convert .sam to .bam + sort .bam
FORWARD=/home/ubuntu/barbara/analysis/trimmed_IlluminaReads/genomePlant/$NAME1\.qc.R1.paired.fastq
REVERSE=/home/ubuntu/barbara/analysis/trimmed_IlluminaReads/genomePlant/$NAME1\.qc.R2.paired.fastq
OUTPREFIX=bov_illumina_on_bov

echo "Aligning $NAME with bwa"
/home/ubuntu/giacomo-2/programs/bwa-0.7.17/bwa mem -M -t $SLURM_CPUS_PER_TASK $INDEX $FORWARD $REVERSE | ~/giacomo-2/programs/samtools/samtools view -@ $SLURM_CPUS_PER_TASK -Sb | ~/giacomo-2/programs/samtools/samtools sort -@ $SLURM_CPUS_PER_TASK -T ${NAME1} > $OUTPREFIX.bam
echo "Stats for $NAME.bwa"
java -jar /home/ubuntu/giacomo-2/programs/picard.jar CollectAlignmentSummaryMetrics \
I=$OUTPREFIX.bam \
R=$REFERENCE \
METRIC_ACCUMULATION_LEVEL=SAMPLE \
O=$OUTPREFIX.alnStats.txt
echo "Index BAM"
~/giacomo-2/programs/samtools/samtools index $OUTPREFIX.bam

# Remove duplicates

echo "Remove duplicates"
java -Xmx2g -jar $PROGRAMS/picard.jar \
MarkDuplicates \
REMOVE_DUPLICATES=true \
ASSUME_SORTED=true \
VALIDATION_STRINGENCY=SILENT \
MAX_FILE_HANDLES_FOR_READ_ENDS_MAP=1000 \
INPUT=$OUTPREFIX.bam \
OUTPUT=$OUTPREFIX.rmd.bam \
METRICS_FILE=$OUTPREFIX.rmd.bam.metrics

echo "Index BAM without duplicates"
~/giacomo-2/programs/samtools/samtools index $OUTPREFIX.rmd.bam

# Variant calling

echo "Variant calling with BCFtools"
$PROGRAMS/bcftools-1.8/bcftools mpileup \
--threads $SLURM_CPUS_PER_TASK \
-a AD,DP,SP -Ou \
-f $REFERENCE $OUTPREFIX.rmd.bam \
| $PROGRAMS/bcftools-1.8/bcftools call \
--threads $SLURM_CPUS_PER_TASK \
-f GQ,GP -m -O z \
-o $OUTPREFIX.vcf.gz

##then need to index
echo "Finished pileup and calling. Now index"
$PROGRAMS/bcftools-1.8/bcftools index $OUTPREFIX.vcf.gz

##get stats: quality and depth
echo "Get VCF stats"
$VCFTOOLS_PATH/vcftools --gzvcf  $OUTPREFIX.vcf.gz\
--site-quality
$VCFTOOLS_PATH/vcftools --gzvcf $OUTPREFIX.vcf.gz \
--site-mean-depth
```
- Mapped illumina reads that were trimmed can thus be found here: `/home/ubuntu/barbara/analysis/vcf/boveana/`.
- To check the vcf file `bcftools view bov_illumina_on_bov.vcf.gz |less`

## Filter 1
Remove indels, sites in repetitive regions, and with quality < 30 and generate statistics.

- First, make a BED file for the repetitive regions:
```
$ cd /home/ubuntu/barbara/analysis/annotation/Pboveana/Repeat_annotation_Final/rep_masker
$ python3 -m jcvi.formats.gff bed --type=similarity --key=Target pbov_polca.filt.fasta.out.gff -o pbov_polca.filt.fasta.out.bed
$ cat pbov_polca.filt.fasta.out.bed | tr ' ' '\t' | sed 's/Motif://g' | cut -f 1-4,7,8 > tmp; mv tmp pbov_polca.filt.fasta.out.bed
```
- Do the filtering with ~/giacomo-3/pedelbergii/vcf/run_vcftools_filter01.sh:
```
cd /home/ubuntu/barbara/analysis/vcf/boveana/genome_plant
cp ~/giacomo-3/pedelbergii/vcf/run_vcftools_filter01.sh .
```
```
echo $(date)
echo $HOSTNAME

VCFTOOLS_PATH=/home/ubuntu/exe_primrose/vcftools/src/cpp
PROGRAMS=/home/ubuntu/my_data/Narcis/executables
BGZIP_PATH=/home/ubuntu/giacomo-2/programs/htslib/

VCF_IN=bov_illumina_on_bov.vcf.gz

NAME=$(basename $VCF_IN .vcf.gz)
VCF_OUT=$NAME\.noIndels.noRep.minQ30.vcf.gz

$VCFTOOLS_PATH/vcftools --gzvcf $VCF_IN \
--exclude-bed ~/barbara/analysis/annotation/Pboveana/Repeat_annotation_Final/rep_masker/pbov_polca.filt.fasta.out.bed \
--remove-indels \
--minQ 30 \
--recode --stdout --out $NAME \
| $BGZIP_PATH/bgzip > $VCF_OUT

$PROGRAMS/bcftools-1.8/bcftools index $VCF_OUT
```
- get statistics
```
$ cp /home/ubuntu/barbara/analysis/vcf/boveana/genome_plant/run_vcftools_stats.sh .
```
```
echo $(date)
echo $HOSTNAME

VCFTOOLS_PATH=/home/ubuntu/exe_primrose/vcftools/src/cpp
PROGRAMS=/home/ubuntu/my_data/Narcis/executables
BGZIP_PATH=/home/ubuntu/giacomo-2/programs/htslib/

VCF_IN=bov_illumina_on_bov.noIndels.noRep.minQ30.vcf.gz
NAME=$(basename $VCF_IN .vcf.gz)

$VCFTOOLS_PATH/vcftools --gzvcf $VCF_IN \
--site-quality
$VCFTOOLS_PATH/vcftools --gzvcf $VCF_IN \
--site-mean-depth


for i in out*;
do mv $i $NAME.$i;
done
```
- Estimate mean coverage
```
$ cp /home/ubuntu/giacomo-3/pedelbergii/vcf/vcf_stats_plots2.R .
$ cp /home/ubuntu/giacomo-3/pedelbergii/vcf/run_plot_vcfstats.sh .
```
Modifie R script and bash file
```
depthFile=ped_illumina_on_ped.noIndels.noRep.minQ30.out.ldepth.mean
qualFile=ped_illumina_on_ped.noIndels.noRep.minQ30.out.lqual

/home/ubuntu/etienne/progs/R-3.6.3/bin/Rscript ./vcf_stats_plots2.R $depthFile $qualFile
```
- Get statistics and plots of the VCF, to chose a threshold for coverage. Coverage distribution is centered at ~140x (P. edelbergii ~160x), while quality is above 250 for most sites(same as for P. edelbergii).
## Filter 2
Based on the coverage (100x-200x) and quality (> 200) thresholds chosen, filter the VCF with `/home/ubuntu/barbara/analysis/vcf/boveana/genome_plant/run_vcftools_filter02.sh`
```
echo $(date)
echo $HOSTNAME

VCFTOOLS_PATH=/home/ubuntu/exe_primrose/vcftools/src/cpp
PROGRAMS=/home/ubuntu/my_data/Narcis/executables
BGZIP_PATH=/home/ubuntu/giacomo-2/programs/htslib/

VCF_IN=bov_illumina_on_bov.noIndels.noRep.minQ30.vcf.gz

NAME=$(basename $VCF_IN .vcf.gz)
VCF_OUT=bov_illumina_on_bov.noIndels.noRep.minQ200.dp100-200.vcf.gz

$VCFTOOLS_PATH/vcftools --gzvcf $VCF_IN \
--min-meanDP 100 \
--max-meanDP 200 \
--minQ 200 \
--recode --stdout --out $NAME \
| $BGZIP_PATH/bgzip > $VCF_OUT

$PROGRAMS/bcftools-1.8/bcftools index $VCF_OUT
```
## Filter 3
For plotting the heterozygosity, filter for only biallelic, heterozygous SNPs. Actually, filter only by max depth! Rerun `run_vcftools_filter02` as following:
```
echo $(date)
echo $HOSTNAME

VCFTOOLS_PATH=/home/ubuntu/exe_primrose/vcftools/src/cpp
PROGRAMS=/home/ubuntu/my_data/Narcis/executables
BGZIP_PATH=/home/ubuntu/giacomo-2/programs/htslib/

VCF_IN=bov_illumina_on_bov.noIndels.noRep.minQ30.vcf.gz

NAME=$(basename $VCF_IN .vcf.gz)
VCF_OUT=bov_illumina_on_bov.noIndels.noRep.minQ200.maxdp200.vcf.gz

$VCFTOOLS_PATH/vcftools --gzvcf $VCF_IN \
#--min-meanDP 100 \
--max-meanDP 200 \
--minQ 200 \
--recode --stdout --out $NAME \
| $BGZIP_PATH/bgzip > $VCF_OUT

$PROGRAMS/bcftools-1.8/bcftools index $VCF_OUT 
```
then run `run_vcftools_filter03` to grep only biallelic, heterozygous SNPs.
```
echo $(date)
echo $HOSTNAME

VCFTOOLS_PATH=/home/ubuntu/exe_primrose/vcftools/src/cpp
PROGRAMS=/home/ubuntu/my_data/Narcis/executables
BGZIP_PATH=/home/ubuntu/giacomo-2/programs/htslib/

VCF_IN=bov_illumina_on_bov.noIndels.noRep.minQ200.maxdp200.vcf.gz
VCF_OUT=bov_illumina_on_bov.noIndels.noRep.minQ200.maxdp200.onlyHet.vcf.gz

zcat $VCF_IN | grep "^#" > $VCF_OUT
zcat $VCF_IN | grep "0/1" >> $VCF_OUT

$PROGRAMS/bcftools-1.8/bcftools index $VCF_OUT
```
## Select Chromosome 2
Run `vcftools_subset.sh` to select chromosome 2.
```
echo $(date)
echo $HOSTNAME


VCFTOOLS_PATH=/home/ubuntu/exe_primrose/vcftools/src/cpp
BGZIP_PATH=/home/ubuntu/giacomo-2/programs/htslib/

VCF_IN=bov_illumina_on_bov.noIndels.noRep.minQ200.dp100-200.vcf.gz
NAME=$(basename $VCF_IN .vcf.gz)
VCF_OUT=bov_illumina_on_bov.noIndels.noRep.minQ200.dp100-200.Chr2.vcf.gz

$VCFTOOLS_PATH/vcftools --gzvcf $VCF_IN --chr pbov_002 \
--recode --stdout --out ${NAME} | $BGZIP_PATH/bgzip > $VCF_OUT
#

#$PROGRAMS/bcftools-1.8/bcftools index $VCF_OUT
```
*ditto* for `bov_illumina_on_bov.noIndels.noRep.minQ200.maxdp200.vcf.gz`

## Plotting heterozygosity across the genome
CMplot
- coverage file
```
$ zcat bov_illumina_on_bov.noIndels.noRep.minQ200.dp100-200.Chr2.vcf.gz | cut -f 1,2,10 | tr ":" "\t" | tr ',' '\t' | cut -f 1,2,4 > bov_illumina_on_bov.noIndels.noRep.minQ200.dp100-200.Chr2.cov.tsv
```
*ditto* for `bov_illumina_on_bov.noIndels.noRep.minQ200.maxdp200.Chr2.vcf.gz`

- heterozygous SNPs file

first `gzip`the `bov_illumina_on_bov.noIndels.noRep.minQ200.maxdp200.onlyHet.vcf.gz` file, as filter03 output is a non-zipped file.
```
$ zcat bov_illumina_on_bov.noIndels.noRep.minQ200.maxdp200.onlyHet.vcf.gz | grep -v "^#" | cut -f 1,2 | awk 'OFS="\t" {print NR,$0}' > bov_illumina_on_bov.noIndels.noRep.minQ200.maxdp200.onlyHet.cmplot.txt
