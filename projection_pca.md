Make a txt file with each ancient sample (ped and map) to be projected:

path_to_sample1/sample1.ped path_to_sample1/sample1.map [space seperated]
path_to_sample2/sample2.ped path_to_sample2/sample2.map

Now merge with the modern dataset. We can use pseudohaploid data projected on a diploid dataset:
```
i="april2017_adM_lab-goats"
dataset="/bowie/adaptmap/version2_of_dataset/ADAPTmap_updated_reduced_cleanedNX0_lab-moderns"

plink --noweb --cow --file $dataset \
--merge-list ancients.txt --recode --out $i

cp "$i".map "$i".pedsnp
cut -f1-6 -d" " "$i".ped > "$i".pedind
echo -e "genotypename: $i.ped\nsnpname: $i.pedsnp\nindivname: $i.pedind\noutputformat: EIGENSTRAT\ngenooutfilename: $i.eigenstratgeno\nsnpoutfilename: $i.snp\nindoutfilename: $i.ind\nnumchrom: 29\noutliermode: 2" > "$i".par

smartpca convertf -p "$i".par > conversion.log #now convert to eigenstrat format
cut -f 1 -d':' "$i".ind > file.pop
awk '{print $1"\t"$2"\t"}' "$i".ind > tmp.txt
paste tmp.txt file.pop > "$i"_alt.ind
cut -f 1 -d" " "$i".pedind | sort -u > populations.txt ##IMTPORTANT: remove all ancients from this file
```
e.g.
```
grep -v "yoqneam*\|pottern*\|kazbeg*\|fars*\|lur*\|azer*\|david*\|qazvin*\|azer*\|semnan*\|direkli*\|acem*\|blagotin*\|ainghazal*\|bulak*\|darre*\|hovk1*\|monjukli*\|AP45*\|AP49*\|geor*\|ghosh*\|yarmu*\|chalow*\|gilat*\|kohneh*\|Kov5*\|miqne*\|safi*\|shiqmim*\|Tac3*" populations.txt  | more > tmp; mv tmp populations.txt
```
Now run the projection using the altered indiv file:
```
echo -e "genotypename: $i.eigenstratgeno\nsnpname: $i.snp\nindivname: "$i"_alt.ind\nevecoutname: "$i"_projection.evec\nevaloutname: $i.eval\npoplistname: populations.txt\nnumoutlieriter: 0\nkillr2: YES\nr2thresh: 0.2\nlsqproject:  YES\nchrom: 29" > "$i"_projection.par
smartpca -p "$i"_projection.par >> projection.log
```
Now drawing the pca:
```
library(ggplot2)
library(rPCA)

plot_colours<- c("blue1", "blue4", "blueviolet", "brown", "brown4", "burlywood4", "cadetblue", "cadetblue4", "chocolate", "chocolate2", "chocolate4", "coral4", "cornflowerblue", "darkgoldenrod", "darkgreen", "darkorchid1", "darkorchid4", "darkred", "darkslateblue", "darkslategrey", "deeppink", "deeppink4", "deepskyblue", "deepskyblue4", "firebrick", "forestgreen", "hotpink", "hotpink4", "indianred", "indianred4","khaki4", "mediumpurple", "mediumpurple4", "mediumseagreen", "olivedrab", "orange3", "plum4", "royalblue4", "palevioletred4", "salmon4","saddlebrown", "sienna", "slateblue4", "turquoise4", "springgreen4", "slateblue1", "tomato", "thistle4", "violetred", "wheat4", "yellow,")

dat<-read_evec("sample_projection_evec.txt")

dat$ancients <- ifelse(grepl("[ABCDEFGHIJKLMNOPQRSTUVWXYZ]",dat$pop, perl=TRUE), "modern", "ancient") #if necessary

dat[grep("ainghazal", dat$pop),]$ancients <- "neolithic_jordan"
dat[grep("blagotin", dat$pop),]$ancients <- "neolithic_serbia" #add appropriate label
dat[grep("direkli", dat$pop),]$ancients <- "epipaleolithic_turkey" #add appropriate label
dat[grep("acem", dat$pop),]$ancients <- "bronze_turkey"
dat[grep("qazvin1", dat$pop),]$ancients <- "chalcolithic_iran"
dat[grep("azer4", dat$pop),]$ancients <- "iron_iran"
dat[grep("azer3-5", dat$pop),]$ancients <- "bronze_iran" 
dat[grep("azer6", dat$pop),]$ancients <- "chalcolithic_iran" 
dat[grep("semnan", dat$pop),]$ancients <- "neolithic_iran"
dat[grep("david", dat$pop),]$ancients <- "iron_israel"
dat[grep("kazbeg", dat$pop),]$ancients <- "medieval_georgia"
dat[grep("geor", dat$pop),]$ancients <- "medieval_georgia"
dat[grep("fars4", dat$pop),]$ancients <- "chalcolithic_iran"
dat[grep("fars1", dat$pop),]$ancients <- "chalcolithic_iran"
dat[grep("blag", dat$pop),]$ancients <- "neolithic_serbia"
dat[grep("yoqneam", dat$pop),]$ancients <- "bronze_israel"
dat[grep("lur", dat$pop),]$ancients <- "neolithic_iran"
dat[grep("pottern", dat$pop),]$ancients <- "bronze_britain"
dat[grep("monjukli8", dat$pop),]$ancients <- "neolithic_turkmen"
dat[grep("monjukli1", dat$pop),]$ancients <- "chalcolithic_turkmen"
dat[grep("monjukli2", dat$pop),]$ancients <- "chalcolithic_turkmen"
dat[grep("monjukli4", dat$pop),]$ancients <- "chalcolithic_turkmen"
dat[grep("darre", dat$pop),]$ancients <- "chalcolithic_iran"
dat[grep("hovk1", dat$pop),]$ancients <- "epipaleolithic_armenia"
dat[grep("bulak", dat$pop),]$ancients <- "bronze_uzbek"
dat[grep("IOG", dat$pop),]$ancients <- "modern_ireland"
dat[grep("Tog", dat$pop),]$ancients <- "modern_toga"
dat[grep("gilat", dat$pop),]$ancients <- "chal_israel" 
dat[grep("safi", dat$pop),]$ancients <- "bronze_israel"
dat[grep("miqne", dat$pop),]$ancients <- "iron_israel"
dat[grep("Tac3", dat$pop),]$ancients <- "bronze_georgia" 
dat[grep("shiqmim", dat$pop),]$ancients <- "chal_israel"
dat[grep("kohneh", dat$pop),]$ancients <- "bronze_iran"
dat[grep("chalow", dat$pop),]$ancients <- "bronze_iran"
dat[grep("AP4", dat$pop),]$ancients <- "neolithic_turkey" 
dat[grep("fars2-5", dat$pop),]$ancients <- "neolithic_iran" 
dat[grep("yarmut", dat$pop),]$ancients <- "bronze_israel"

#subsample 20 from each population
n<-20
FUN <- function(x, n) {
    if (length(x) <= n) return(x)
    x[x %in% sample(x, n)]
    }
reduced_dat<-dat[unlist(lapply(split(1:nrow(dat), dat$pop), FUN, n = 20)), ]
pca <-ggplot(data=reduced_dat, aes(x=PC1,y=PC2, color=ancients, label=pop)) + geom_text(size=2,alpha=0.8) + theme_bw()  + scale_colour_manual(values=c("purple3", "springgreen4", "chocolate4", "sienna", "orange", "indianred","turquoise4", "red", "coral","slateblue2", "red", "cadetblue", "darkgrey", "plum4", "tomato", "springgreen4", "slateblue1", "violetred", "wheat4", "yellow2", "springgreen1", "orange", "plum8" , "slateblue4", "springgreen7", "yellow5"),name="Context", breaks=c("modern","epipaleolithic_armenia","epipaleolithic_turkey","neolithic_iran","neolithic_turkmen","neolithic_turkey","neolithic_serbia","neolithic_jordan","chalcolithic_iran", "chalcolithic_turkmen","chalcolithic_israel","bronze_iran","bronze_georgia","bronze_turkey", "bronze_uzbek","bronze_israel", "bronze_britain","iron_iran", "iron_israel","medieval_georgia","modern_ireland","modern_toga" ),labels=c("Modern","Epipaleolithic Armenia","Epipaleolithic Turkey","Neolithic Iran","Neolithic Turkmenistan","Neolithic Turkey","Neolithic Serbia", "Neolithic Jordan","Chalcolithic Iran","Chalcolithic Turkmenistan","Chalcolithic Israel","Bronze Iran","Bronze Georgia","Bronze Age Turkey",  "Bronze Age Uzbekistan" ,  "Bronze Age Israel", "Bronze Age Britain", "Iron Age Iran", "Iron Age Israel", "Medieval Georgia","Modern Ireland", "Modern Toga"))  + guides(label=FALSE)
pca
pdf("pca.pdf", width = 35, height = 35, paper="a4r")
dev.off()

```
