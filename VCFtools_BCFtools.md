NOTE: When filtering the option --mac 2 eliminates singletons so no need to apply the --maf 0.XXX flag.


In order to filter some GQ  sites I need to change some parts with sed

```
bcftools filter -i 'QUAL>998 & MIN(GQ)>98' -r pve_haplotypeT_001:27600446-27649965 merged_allpops_s_locus_filtered_transcripts_norepeats.vcf.gz | grep -v '^#' | tail -1 | sed -e 's/,.:.:.:./,0:0:0:99/g' | sed -e 's/.:.:.:.:.:.:./.:.:.:.:.:.:0/g'
```

Select sites with QUALITY above 998 and from CYPt

```
bcftools filter -i 'QUAL>998' -r pve_haplotypeT_001:27600446-27649965 merged_allpops_s_locus_filtered_transcripts_norepeats.vcf.gz | sed -e 's/,.:.:.:./,0:0:0:99/g' | sed -e 's/.:.:.:.:.:.:./.:.:.:.:.:.:0/g' 
```

In the alignment there are some "dubious" sites  

```
vcftools --gzvcf merged_allpops_s_locus_filtered_transcripts_norepeats_cypt.vcf.gz --recode --recode-INFO-all --stdout --minGQ 99 | bgzip -c > merged_allpops_s_locus_filtered_transcripts_norepeats_cypt_filtered.vcf.gz
bcftools index merged_allpops_s_locus_filtered_transcripts_norepeats_cypt_filtered.vcf.gz
```

```
bcftools filter -i 'QUAL>998' merged_allpops_s_locus_filtered_transcripts_norepeats.vcf.gz | sed -e 's/,.:.:.:./,0:0:0:99/g' | sed -e 's/.:.:.:.:.:.:./.:.:.:.:.:.:0/g'  | bgzip -c > merged_allpops_s_locus_filtered_transcripts_norepeats_corrected.vcf.gz
bcftools index merged_allpops_s_locus_filtered_transcripts_norepeats_corrected.vcf.gz
```

```
bcftools query -f '%POS %REF %ALT [%GT:%AD:%GQ ]\n' merged_allpops_s_locus_filtered_transcripts_norepeats.vcf.gz
```

vcftools --gzvcf merged_allpops_s_locus_filtered_transcripts_norepeats_corrected.vcf.gz --recode --recode-INFO-all --stdout --minGQ 99 | bgzip -c > merged_allpops_s_locus_filtered_transcripts_norepeats_corrected_GCfiltered.vcf.gz

bcftools index merged_allpops_s_locus_filtered_transcripts_norepeats_corrected_GCfiltered.vcf.gz
---
Problem:
- Homozygous calls for the reference allele produce no Genotype Quality (GQ) score when the variant calling is done with BCFtools. This is a problem because when doing a filter with VCFtools sites with no GQ score are turned into no call.
Solution:
- Asign GQ score myself using sed.
```
bcftools filter -i 'QUAL>998' merged_allpops_s_locus_filtered_transcripts_norepeats.vcf.gz | \
sed -e 's/,.:.:.:./,0:0:0:99/g' | sed -e 's/.:.:.:.:.:.:./.:.:.:.:.:.:0/g' | \
bgzip -c > merged_allpops_s_locus_filtered_transcripts_norepeats_cypt.vcf.gz
bcftools index merged_allpops_s_locus_filtered_transcripts_norepeats_cypt.vcf.gz #index the file
```

---
Problem:
- There are some sites that have 'fake' heterozygosity when there is Allelic Imbalance. For example when '0/1 | 1, 10' the genotype will be considered a heterozygote, even if only a single base has the reference allele.  
Solution:
- Calculate allellic imbalance (AB) based on the FORMAT/AD and make changes accordingly.
```
#Excludes every site where at least one individual has a higher presence of of the REF allele (i.e., AD[ALT] / AD[REF] < 0.33).

bcftools filter -e 'GT="het" & (FORMAT/AD[*:1])/(FORMAT/AD[*:0]) <= 0.33' EM33.vcf.gz -Ov -o test_2.vcf.gz

#Excludes every site where at least one individual has a higher presence of of the REF allele (i.e., AD[ALT] / AD[REF] < 0.33) OR ALT allele (i.e., AD[ALT] / AD[REF] > 0.75).

bcftools filter -e 'GT="het" & (FORMAT/AD[*:1])/(FORMAT/AD[*:0]) <= 0.33 | GT="het" & (FORMAT/AD[*:1])/(FORMAT/AD[*:0]) >= 0.75 ' EM33.vcf.gz -Ov -o test_3.vcf.gz

# The above commands will eliminate every site where at least one sample has does filters, thus loosing a lot of sites. What we can do is to identify those genotypes and changen them to no call ('./.').
bcftools filter -e 'GT="het" & (FORMAT/AD[*:1])/((FORMAT/AD[*:0]) + (FORMAT/AD[*:1]))  <= 0.33 | GT="het" & (FORMAT/AD[*:1])/((FORMAT/AD[*:0]) + (FORMAT/AD[*:1])) >= 0.75 ' EM33.vcf.gz --set-GTs . -Oz -o test_5.vcf.gz

# Still trying to figure out how can I make some sites to be homozygpus for the REF allele and how I can make some others to be hmozygous for the ALT allel.
bcftools filter -e 'GT="het" & (FORMAT/AD[*:1])/((FORMAT/AD[*:0]) + (FORMAT/AD[*:1]))  <= 0.33' EM33_biallelic_norepeats_notranscripts_nocentromeres_001.vcf.gz --set-GTs 0 | bcftools filter -e 'GT="het" & (FORMAT/AD[*:1])/((FORMAT/AD[*:0]) + (FORMAT/AD[*:1]))  >= 0.75' EM33_biallelic_norepeats_notranscripts_nocentromeres_001.vcf.gz --set-GTs 1 -Oz -o test_6.vcf.gz
```

---

```
for i in ${pops[@]}; do
$BCFTOOLS/bcftools view -h $DIR/$i/marked_dup_BAMs/s_locus/hq_s_locus/''$i'_slocus_hq.vcf.gz' > header_$i.txt # Extracts header from VCF file and sends it to txt file
sed 's/^##FORMAT=<ID=AD,Number=R/##FORMAT=<ID=AD,Number=./' header_$i.txt > 'header_'$i'_fixed.txt' # Substitutes Number=R to Number=. otherwise bcftools does not run and creates a new txt file
sed -E 's/(W\w[0-12]*[0-9])/'$i'_\1/g' 'header_'$i'.txt' > 'header_'$i'_fixed.txt' # Adds the population ID to each individual in the header and creates a new txt file
$BCFTOOLS/bcftools reheader -h 'header_'$i'_fixed.txt' -o ''$i'_s_locus_header_fixed.vcf.gz' $DIR/$i/marked_dup_BAMs/s_locus/hq_s_locus/''$i'_slocus_hq.vcf.gz' # Add the new header into the VCF file
$BCFTOOLS/bcftools index ''$i'_s_locus_header_fixed.vcf.gz'
rm *.txt;done # Index the new VCF file
```

To show a VCF file into column format
```
zcat merged_allpops_s_locus_filtered.vcf.gz | grep -v "^##" | less -S

#Print column 4 (REF allele) of the VCF
zcat merged_allpops_s_locus_filtered.vcf.gz | grep -v "^##" | less -S | awk '{print $4}'

# Paste the column 4 into a file and make it a string
zcat merged_allpops_s_locus_filtered.vcf.gz | grep -v "^##" | less -S | awk '{print $4}' | paste -s -d "" > reference
```

Filter VCF file based on quality
```
vcftools --gzvcf merged_allpops_s_locus_filtered_transcripts_norepeats.vcf.gz --chr pve_haplotypeT_001 --from-bp 27600446 --to-bp 27649965 --recode --recode-INFO-all --stdout --minQ 999 | bgzip -c  > merged_allpops_s_locus_filtered_transcripts_norepeats_cypt.vcf.gz

#equivalent on BCFtools

bcftools filter -i 'QUAL>998 && GQ>98' -r pve_haplotypeT_001:27600446-27649965 merged_allpops_s_locus_filtered_transcripts_norepeats.vcf.gz | grep -v '^#' |  grep 27638819
```