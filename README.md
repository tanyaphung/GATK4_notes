# GATK4_notes
Notes on GATK4. Here, I'm interested in calling variants and outputing all of the sites (not just the variants). 

### 1. Call variants per sample per chromosome to generate the gVCF files
* Tool: `HaplotypeCaller`
* --emit-ref-confidence BP_RESOLUTION

```
gatk HaplotypeCaller \
  -R ref \
  -I bam \
  -L chrm \
	--emit-ref-confidence BP_RESOLUTION \
  --output output
```

### 2. Consolidate GVCFs

* Tools: `GenomicsDBImport` or `CombineGVCFs`.

* The GATK4 Best Practice Workflow for SNP and Indel calling uses GenomicsDBImport to merge GVCFs from multiple samples. GATK highly recommends using `GenomicsDBImport` for consolidating (over `CombineGVCFs`). However, I find that when using `GenomicsDBImport` to consolidate GVCFs, the flag `--include-non-variant-sites` does not work in the `GenotypeGVCFs` step. If you do not need to include the non-variant sites in your VCF, I find that `GenomicsDBImport` and `GenotypeGVCFs` work well. 

* As mentioned above, I find that in order for the flag `--include-non-variant-sites` to work in the `GenotypeGVCFs` step, the input consolidated GVCF file should be consolidated using the tool `CombineGVCFs` (rather than `GenomicsDBImport`). 

* Using `GenomicsDBImport`:
```
gatk --java-options '-Xmx10g' CombineGVCFs \
-R ref \
--variant sample1.g.vcf.gz \
--variant sample2.g.vcf.gz \
--output all.g.vcf.gz
```
* Using `CombineGVCFs`:
```
gatk --java-options -Xmx8g \
GenomicsDBImport \
--genomicsdb-workspace-path my_database 
-L chrm 
--sample-name-map cohort_sample_map
--tmp-dir=tmp_dir 
--reader-threads 5
```
  - The file `cohort_sample_map` is a tab-separated file where the first column is the name of the sample and the second column is the path to the GVCF file for that sample. Note that this has to be tab-separated or else it will fail. 

### 3. Joint Genotyping

* Tool: `GenotypeGVCFs`

```
gatk --java-options '-Xmx8g' GenotypeGVCFs \
-R ref \
-V all.g.vcf.gz \
--include-non-variant-sites \
--output allsites.vcf
```
