










Convert pileup file to homozygous ped file
```
for i in $(ls *.pileup | cut -f1 -d'.');\
do \
~/bin/pileupTools/pileupTools.py "$i".pileup -s $i > "$i"_HOM.log; \
ped_hom "$i".ped > "$i"_HOM.ped; \
done
```



Merge homozygous samples and homozygous dataset

```
for i in $(ls *.pileup | cut -f1 -d'.'); \
do \
cp "$i".map "$i"_HOM.map; \
plink --cow --file /bowie/adaptmap/version1/adaptmapTOP-HOM-matthew-recoded-finalv1-filt2 \
--merge $i"_HOM" --recode --out "$i"_merged_HOM;\
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



Drawing pca in R
```
library(rPCA)
dat<-read_evec("sample_evec.txt")


dat$ancients<-ifelse(grepl("[ABCDEFGHIJKLMNOPQRSTUVWXYZ]",dat$pop, perl=TRUE), "modern", "ancient")  #assumes pops are CAP

tmp <- dat[order(dat$pop),] #Sort by population, subsample 20 from each
d <- by(tmp, tmp["pop"],n=4)
reduced_dat<-Reduce(rbind, d)


pca <-ggplot(data=dat, aes(x=PC1,y=PC2, color=ancients, label=pop))  + geom_text(size=2,alpha=0.8) + theme_bw()  + scale_colour_manual(values=c("coral",  "darkgrey"))  + theme(legend.position="none") #colours ancients, greys moderns

```
OR
```
pca <-ggplot(data=dat, aes(x=PC1,y=PC2, color=pop, label=pop))  + geom_text(size=2) + scale_shape_manual(values=rep(c(0:10,12:25,48:57,65:122),18)) + theme_bw()  + scale_colour_manual(values=rep(plot_colours,10))  + theme(legend.position="none")

pca

```

dat$ancients<-ifelse(grepl("[ABCDEFGHIJKLMNOPQRSTUVWXYZ]",dat$pop, perl=TRUE), "modern", "ancient")  
