Make a txt file with each ancient sample (ped and map) to be projected:

path_to_sample1/sample1.ped path_to_sample1/sample1.map
path_to_sample2/sample2.ped path_to_sample2/sample2.map

Now merge with the modern dataset. Assuming we use a pseudohaploidized dataset when using pseudohaploid data:
```
i="projection_test"
dataset="/bowie/adaptmap/version1/adaptmapTOP-recoded-finalv1-filt2"

plink --noweb --cow --file $dataset \
--merge-list ancients.txt --recode --out $i

cut -f1-6 -d" " "$i".ped > "$i".pedind
cp "$i".map "$i".pedsnp
cut -f1-6 -d" " "$i".ped > "$i".pedind
echo -e "genotypename: $i.ped\nsnpname: $i.pedsnp\nindivname: $i.pedind\noutputformat: EIGENSTRAT\ngenooutfilename: $i.eigenstratgeno\nsnpoutfilename: $i.snp\nindoutfilename: $i.ind" > "$i".par

smartpca convertf -p "$i".par > conversion.log #now convert to eigenstrat format
cut -f 1 -d':' "$i".ind > file.pop
awk '{print $1"\t"$2"\t"}' "$i".ind > tmp.txt
paste tmp.txt file.pop > "$i"_alt.ind
cut -f 1 -d" " "$i".pedind | sort -u > populations.txt ##IMTPORTANT: remove all ancients from this file
```
e.g.
```
grep -v "qazvin*\|azer*\|semnan*\|direkli*\|acem*\|blagotin*\|ainghazal*" populations.txt  | more > tmp; mv tmp populations.txt
```
Now run the projection using the altered indiv file:
```
echo -e "genotypename: $i.eigenstratgeno\nsnpname: $i.snp\nindivname: "$i"_alt.ind\nevecoutname: "$i"_projection.evec\nevaloutname: $i.eval\npoplistname: populations.txt\nnumoutlieriter: 0\nkillr2: YES\nr2thresh: 0.2\nlsqproject:  YES" > "$i"_projection.par
smartpca -p "$i"_projection.par >> projection.log
```
Now drawing the pca:
```
library(ggplot2)
library(rPCA)

dat<-read_evec("sample_projection_evec.txt")

dat$ancients <- ifelse(grepl("[ABCDEFGHIJKLMNOPQRSTUVWXYZ]",dat$pop, perl=TRUE), "modern", "ancient") #if necessary

dat[grep("ainghazal", dat$pop),]$ancients <- "neolithic_turkey" #add appropriate label

#subsample 20 from each population
n<-20
FUN <- function(x, n) {
    if (length(x) <= n) return(x)
    x[x %in% sample(x, n)]
    }
reduced_dat<-dat[unlist(lapply(split(1:nrow(dat), dat$pop), FUN, n = 20)), ]

pca <-ggplot(data=reduced_dat, aes(x=PC1,y=PC2, color=ancients, label=pop))  + geom_text(size=2,alpha=0.8) + theme_bw()  + scale_colour_manual(values=c("coral", "darkgrey", "red", "purple3", "springgreen4", "chocolate4", "sienna"),name="Context", breaks=c("bronze_turkey","modern","neolithic_iran","neolithic_serbia","neolithic_turkey","paleolithic_turkey"),labels=c("Bronze Age Turkey","Modern","Neolithic Iran", "Neolithic Serbia", "Neolithic Turkey", "Epipaleolithic Turkey"))  + guides(label=FALSE)

pca
```

```
