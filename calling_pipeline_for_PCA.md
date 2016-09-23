See Victoria's instructions for preparing modern dataset for use.

Call a high coverage sample (8X)
```
DUMMY="/bowie/adaptmap/version2/vcfout_header-added_formated.vcf"
TABLE="/bowie/adaptmap/version2/metatable.txt"
INTERVALS="/bowie/adaptmap/version2/ADAPTmap_updated_reduced_cleanedNX0.interval_list"
REFERENCE="/kendrick/reference_genomes/goat_CHIR1_0/goat_CHIR1_0.fasta"
DATASET="/bowie/adaptmap/version2/ADAPTmap_HOM_updated_reduced_cleanedNX0"

for SAMPLE in a b c; do \
java -Xmx2g -jar /home/admin1/Software/GATK/GenomeAnalysisTK.jar \
-R $REFERENCE \
-I $SAMPLE.bam \
-o $SAMPLE.vcf \
-T UnifiedGenotyper \
-L $INTERVALS \
-gt_mode GENOTYPE_GIVEN_ALLELES -mbq 20 -out_mode EMIT_ALL_SITES \
--alleles $DUMMY \
--dbsnp $DUMMY
```

Low coverage sample - pseudohaploidize the data.
```
 java -Xmx4g -jar /home/admin1/Software/GATK/GenomeAnalysisTK.jar \
-R $REFERENCE \
-T Pileup \
-I $SAMPLE.bam \
-L $INTERVALS
--metadata:TABLE $TABLE
-o $SAMPLE.pileup
```



Convert pileup file to homozygous ped file
```
for SAMPLE in $(ls *.pileup | cut -f1 -d'.');\
do \
~/bin/pileupTools/pileupTools.py "$SAMPLE".pileup -s $SAMPLE > "$SAMPLE"_HOM.log; \
ped_hom "$SAMPLE".ped > "$SAMPLE"_HOM.ped; \
done
```



Merge homozygous samples and homozygous dataset

```
for SAMPLE in $(ls *.pileup | cut -f1 -d'.'); \
do \
cp "$SAMPLE".map "$SAMPLE"_HOM.map; \
plink --cow --file $DATASET \
--merge "$SAMPLE"_HOM --recode --out "$SAMPLE"_merged_HOM;\
plink --cow --file "$SAMPLE"_merged_HOM --geno 0 --recode --out "$SAMPLE"_merged_HOM_geno0;\
done
```
Create inputs for smartpca
```
for SAMPLE in $(ls *geno0.ped | cut -f1 -d'.'); do cp "$SAMPLE".map "$SAMPLE".pedsnp;\
cut -f1-6 "$SAMPLE".ped > "$SAMPLE".pedind;\
echo -e "genotypename: $SAMPLE.ped\nsnpname: $SAMPLE.pedsnp\nindivname: $SAMPLE.pedind\nevecoutname: "$SAMPLE"_evec.txt\nevecoutname "$SAMPLE"_ecal.txt\nkillr2: Yes\nr2thresh: 0.2\nnumchrom: 29" > "$SAMPLE".par;\
smartpca -p "$SAMPLE".par > "$SAMPLE"_SMARTPCA.log & done
```
