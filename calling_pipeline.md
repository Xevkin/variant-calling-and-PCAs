












#Merge homozygous samples and homozygous dataset

```for i in $(ls *pileup | cut -f1 -d'.')
  do
  plink --cow --file /bowie/adaptmap/version1/adaptmapTOP-HOM-matthew-recoded-finalv1-filt2 \
  --merge $i --recode --out "$i"_merged_HOM; 
  plink --cow --file "$i"_merged_HOM --geno 0 --recode --out "$i"_merged_HOM_geno0
  done```
