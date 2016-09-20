Make a txt file with each ancient sample (ped and map) to be projected:

path_to_sample1/sample1.ped path_to_sample1/sample1.map
path_to_sample2/sample2.ped path_to_sample2/sample2.map

Now merge with the modern dataset. Assuming we use a pseudohaploidized dataset when using pseudohaploid data:
```
plink --noweb --cow --file /bowie/adaptmap/version1/adaptmapTOP-HOM-matthew-recoded-finalv1-filt2 \
--merge-list projection_test.txt --recode --out projection_test
cut -f1-6 -d" " projection_test.ped > projection_test.pedind


i="projection_test"
cp "$i".map "$i".pedsnp
cut -f1-6 -d" " "$i".ped > "$i".pedind
echo -e "genotypename: $i.ped\nsnpname: $i.pedsnp\nindivname: $i.pedind\noutputformat: EIGENSTRAT\ngenooutfilename: $i.eigenstratgeno\nsnpoutfilename: $i.snp\nindoutfilename: $i.ind" > "$i".par

smartpca convertf -p projection_test.par > conversion.log #now convert to eigenstrat format
```
