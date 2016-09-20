












Merge homozygous samples and homozygous dataset

```
for i in $(ls *pileup | cut -f1 -d'.'); \
do \
plink --cow --file /bowie/adaptmap/version1/adaptmapTOP-HOM-matthew-recoded-finalv1-filt2 \
--merge $i --recode --out "$i"_merged_HOM;\
plink --cow --file "$i"_merged_HOM --geno 0 --recode --out "$i"_merged_HOM_geno0;\
done
```
Create inputs for smartpca
```
for i in $(ls *geno0.ped | cut -f1 -d'.'); do cp "$i".map "$i".pedsnp;\
cut -f1-6 "$i".ped > "$i".pedind;\
echo -e "genotypename: $i.ped\nsnpname: $i.pedsnp\nindivname: $i.pedind\nevecoutname: "$i"_evec.txt\nevecoutname "$i"_ecal.txt\nkillr2: Yes\nr2thresh: 0.2\nnumchrom: 29" > "$i".par;\
smartpca -p "$i".par > "$i"_SMARTPCA.log & done
```



