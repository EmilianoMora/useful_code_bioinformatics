https://gatk.broadinstitute.org/hc/en-us/articles/360037226472-AddOrReplaceReadGroups-Picard-

https://gatk.broadinstitute.org/hc/en-us/articles/360035889471?id=3060

https://gatk.broadinstitute.org/hc/en-us/articles/360035890671-Read-groups

https://gatk.broadinstitute.org/hc/en-us/articles/360037226472-AddOrReplaceReadGroups-Picard-#--RGID

https://gatk.broadinstitute.org/hc/en-us/articles/360035532352-Errors-about-read-group-RG-information



BWA

https://www.biostars.org/p/429329/

https://www.biostars.org/p/269391/

https://www.biostars.org/p/395715/



sed '/@SQ/d;/@HD/d;/@PG/d' WG07_on_PveHaploT.sam | awk '{print $1}' | sed 's/:/\t/g' | awk '{print $3,$4'} | sort -u



The following code will break a BAM files into separate SAM based on the read group (i.e. cell flow and sequencing lane) 

var=$(samtools view -h /home/ubuntu/emiliano/test_bwa/WG08_on_PveHaploT.bam | sed '/@SQ/d;/@HD/d;/@PG/d' | awk '{print $1}' | sed 's/:/\t/g' | awk '{print $3,$4'} | sort -u | sed 's/\s/:/g')

for i in ${var[@]}; do
	samtools view -H /home/ubuntu/emiliano/test_bwa/WG08_on_PveHaploT.bam > $i_WG08_on_PveHaploT.sam \ #create header for the SAM file
	samtools view -h /home/ubuntu/emiliano/test_bwa/WG08_on_PveHaploT.bam | grep $i >> $i_WG08_on_PveHaploT.sam  \
	java -jar /home/ubuntu/giacomo-2/programs/picard.jar AddOrReplaceReadGroups \
	I=$i_WG08_on_PveHaploT.sam \
	O=$i_WG08_RG.bam \
	RGID=$i \
	RGLB=WG08 \
	RGPL=ILLUMINA \
	RGPU=WG08 \
	RGSM=WG08; done


I should end up with several  SAM files (depending on the number of libraries) and the same number of BAM files with the read groups .
