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
cut -f 1 -d':' projection_test.ind > file.pop
awk '{print $1"\t"$2"\t"}' projection_test.ind > tmp.txt
paste tmp.txt file.pop > projection_test_alt.ind
cut -f 1 -d" " projection_test.pedind | sort -u > populations.txt ##IMTPORTANT: remove all ancients from this file
```
e.g.
```
grep -v "semnan*\|direkli*\|acem*\|blagotin*\|ainghazal*" populations.txt  | more > tmp; mv tmp populations.txt
```

