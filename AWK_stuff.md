## Estimate the average of a specific column. Note: this does not take into account the missing data.
```sh
awk '{total +=$3} END {print total/(NR)}' file # where $3 is the number of the column and NR is the number fo records.
awk '{total +=$3} END {print total/(NR - 1)}' file # add the -1 if there is a name in the column.
```

## Estimate the average while taking missing data into account.
```
awk '{sum +=$3; if($3 != "nan"){count+=1}} END {print sum/count}' file # where "nan" should be replaced by however missing data is coded (e.g., NA, NaN, etc.)
```

## Estimate standard deviation.
```sh
awk '{for(i=5;i<=NF;i++) {sum[i] += $i; sumsq[i] += ($i)^2}} END {for (i=1;i<=NF;i++) {printf "%f %f \n", sum[i]/NR, sqrt((sumsq[i]-sum[i]^2/NR)/NR)}}' FILE.txt
```

## Convert data in a column based on data of other column.
```sh
awk 'BEGIN{FS=OFS="\t"} {if($6 <= 10){$3="nan"}{print}}' file # If data on column 6 ($6) is less than or equal to 10 then flip cell in column 3 ($3) to nan.
awk 'BEGIN{FS=OFS="\t"} {if($6 < 10){$3="nan"}{print}}' file # If data on column 6 ($6) is less than 10 then flip cell in column 3 ($3) to nan.

awk '{for(i=1; i<=NF; i++){sum[i] += $i; if($i != "nan"){count[i]+=1}}} END {for(i=1; i<=NF; i++){if(count[i]!=0){v = sum[i]/count[i]else{v = 0}; if(i<NF){printf "%f\t",v}else{print v}}}' file # This one does the same as above but for every column as loop.
```

## To eliminate ALL columns AND rows that are 'nan' (i.e., zero). In the case of the linkage disequilibrium you end up with a squared matrix.  Taken from here.
```sh
awk '{show=0; for (i=1; i<=NF; i++) {if ($i!=0) show=1; col[i]+=$i;}} show==1{tr++; for (i=1; i<=NF; i++) vals[tr,i]=$i; tc=NF} END{for(i=1; i<=tr; i++) { for (j=1; j<=tc; j++) { if (col[j]>0) printf("%s%s", vals[i,j], OFS)} print ""; } }' EM33_report_sq_win.ld | awk '{s=0; for (i=3;i<=NF;i++) s+=$i; if (s!=0)print}'

The output looks like this:

1 0.00185185 0.0680556 0.25 0.25 0.047619 
0.00185185 1 0.592593 0.0625 0.0625 0.107143 
0.0680556 0.592593 1 0.107143 0.107143 0.183673 
0.25 0.0625 0.107143 1 1 0.583333 
0.25 0.0625 0.107143 1 1 0.583333 
0.047619 0.107143 0.183673 0.583333 0.583333 1
```

The above command looks complicated but the first AWK command columns with ALL 'zeros' or 'nan'. Whereas the second AWK command eliminates rows with ALL 'zeros' or 'nan'.
```sh
1)
awk '{show=0; for (i=1; i<=NF; i++) {if ($i!=0) show=1; col[i]+=$i;}} show==1{tr++; for (i=1; i<=NF; i++) vals[tr,i]=$i; tc=NF} END{for(i=1; i<=tr; i++) { for (j=1; j<=tc; j++) { if (col[j]>0) printf("%s%s", vals[i,j], OFS)} print ""; } }' EM33_report_sq_win.ld

2)
awk '{s=0; for (i=3;i<=NF;i++) s+=$i; if (s!=0)print}' EM33_report_sq_win.ld
```

## To transform a space delimited output (example above) into a csv.
```
sed -e 's/\s/,/g' -e 's/   */,/g' file > file.csv
```

## Transform the matrix above into something that looks like a 2d numpy array. However I was not able to load into python. I found the info here and here.
```sh
awk '{print "["$0"]"}' matrix | sed -e 's/\s/,/g' -e 's/   */,/g' | sed 's/,]/],/g' | sed 's/,1],/,1]]/' | sed 's/\[1,/[[1,/'

Output looks like this:

[[1,0.00185185,0.0680556,0.25,0.25,0.047619],
[0.00185185,1,0.592593,0.0625,0.0625,0.107143],
[0.0680556,0.592593,1,0.107143,0.107143,0.183673],
[0.25,0.0625,0.107143,1,1,0.583333],
[0.25,0.0625,0.107143,1,1,0.583333],
[0.047619,0.107143,0.183673,0.583333,0.583333,1]]
```
```sh
awk '{if($1="pve_haplotypeT_001 && $4>27468912) print $0}' /home/ubuntu/emiliano/pve_final-files_backup/gff/pve_haplotypeT_exons.final.gff |less
```

```sh
bcftools query -f '%POS\t%INFO/InbreedingCoeff\n' EM33_snp_indel_novar_flagged_filtered_header_fixed_biallelic_no_indel_001.vcf.gz > inb_coeff

awk '{A[$2]++}END{for(i in A)print i,A[i]}' inb_coeff | awk '{ printf("%.1g %3g\n", $1, $2) }' | awk '{ seen[$1] += $2 } END { for (i in seen) print i, seen[i] }'
```

Problem: concatenate files but keep the file name as a column. Got code from [here](https://unix.stackexchange.com/questions/153773/cat-a-directories-files-apending-the-file-name-to-the-row-of-text-and-removing-t)
```sh
Example
cd  /home/ubuntu/emiliano/2nd_chp/scikit/pi/output/50k_window
awk 'NR<1; FNR>1 {print FILENAME,$0}' *4fold* > all.txt
```

## Add 2000bp downstream of any coding region to the gff
```sh
awk '{$4-=2000}1' pve_haplotypeT_transcript.final.gff | awk '{$5+=2000}1' | awk '{$4=($4<0)?1:$4}1' | awk -v OFS='\t' '{print $1,$2,$3,$4,$5,$6,$7,$8,$9}' | sed '1d' > pve_haplotypeT_transcript_2kb_downstream_2.final.gff
```

## Create new column based on information of other column. Found here.
```
awk 'NR==0{$8="";print;next}\
 $1 == "RS170_4fold_pi" {$8="D"};\
 $1 == "RS180_4fold_pi" {$8="D"};\
 $1 == "RSBK01_4fold_pi" {$8="D"};\
 $1 == "EMC6_4fold_pi" {$8="D"};\
 $1 == "EMC3_4fold_pi" {$8="D"};\
 $1 == "EMC1_4fold_pi" {$8="D"};\
 $1 == "EM32_4fold_pi" {$8="T"};\
 $1 == "EM07_4fold_pi" {$8="T"};\
 $1 == "EM33_4fold_pi" {$8="M"}1' all.txt | column -t > all_pops_4fold.txt
```

## Shuffle lines from this link (https://stackoverflow.com/questions/2153882/how-can-i-shuffle-the-lines-of-a-text-file-on-the-unix-command-line-or-in-a-shel/2153889#2153889)
```sh
shuffle() { 
    awk 'BEGIN{srand();} {printf "%06d %s\n", rand()*1000000, $0;}' | sort -n | cut -c8-
}

USAGE:
any_command | shuffle
```

```
bcftools query -f '%CHROM %POS[\t%DP]\n' FILE.vcf.gz | head | awk '{for(i=1; i<=NF; i++) {a[i]+=$i; if($i!="") b[i]++}}; END {for(i=1; i<=NF; i++) printf "%s%s", a[i]/b[i], (i==NF?ORS:OFS)}'
```

