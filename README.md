# PureCN
## PureCN pipeline for CNV analysis using PureCN docker image

## PureCN Tumor-Normal pair:

### 1. Mount docker with the file path

	for running in drive:
	docker run -it --rm -v /c/Users/drish/OneDrive/Documents/Docs/PureCN/files:/in_bed/ markusriester/purecn:latest

	for running in this pc
	docker run -it --rm -v /mnt/c/Users/drish/Documents/PureCN/Intervals:/in_bed/ markusriester/purecn:latest
	
So, any files or directories present in /c/Users/drish/OneDrive/Documents/Docs/PureCN/files on your host machine will be accessible inside the Docker container at /in_bed/. You can think of /in_bed/ as a sort of "mount point" within the container where the files from your host are mounted.

### 2. Create Interval files from (opt directory):	

	Rscript PureCN/IntervalFile.R --in-file ../in_bed/hg38_Twist_ILMN_Exome_2.5_Panel_annotated.BED --fasta ../in_bed/hg38.fa --out-file ../in_bed/baits_hg38_interval.txt --genome hg38 --export ../in_bed/baits_optimized_hg38.bed --mappability ../in_bed/GCA_000001405.15_GRCh38_no_alt_analysis_set_76.bw


### 3. Create Tumor and Normal coverage files

	Rscript PureCN/Coverage_1.R --out-dir ../in_bed/coverage/Tumor --bam ../in_bed/Intervals/S033.bam --intervals ../in_bed/Intervals/baits_hg38_interval.txt

### 4. Create NormalDB

In case no GC-normalization is performed:
	ls -a ../in_bed/coverage/Normal/*_coverage.txt.gz | cat > normal_coverages.list
	Rscript PureCN/NormalDB.R --out-dir ../in_bed/Normaldb/ --coverage-files normal_coverages.list --genome hg38

### 5. Create a mutect.vcf file if you haven't already

	gatk Mutect2 -R hg38.fa -I S033.bam -L hg38_Twist_ILMN_Exome_2.5_Panel_annotated.BED --f1r2-tar-gz S033_tumor-f1r2.tar.gz -O S033_tumor_variants.vcf

### 6. Run PureCN script

Rscript $PURECN/PureCN.R --out ../in_bed/PureCN_out_1/S033 --tumor ../in_bed/coverage/Tumor/S033_coverage_loess.txt.gz --normal ../in_bed/coverage/Normal/S043_coverage_loess.txt.gz --sampleid S033 --vcf S033_tumor_variants.vcf --normaldb ../in_bed/Normaldb/normalDB_hg19.rds --intervals ../in_bed/baits_hg38_intervals.txt --genome hg38




## Extras in QC:
### 1. Find coverage depth in both tumor and normal to compare thecoverage compatibility:
   
	gatk DepthOfCoverage    -I S033_sorted.bam    -O gatk_coverage    -R hg38.fa    -L Exome.interval_list    --omit-depth-output-at-each-base    --omit-interval-statistics
	gatk DepthOfCoverage    -I S043_sorted.bam    -O gatk_coverage    -R hg38.fa    -L Exome.interval_list    --omit-depth-output-at-each-base    --omit-interval-statistics
	
