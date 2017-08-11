

Drawing pca in R
```
library(rPCA)
dat<-read_evec("sample_evec.txt")


dat$ancients<-ifelse(grepl("[ABCDEFGHIJKLMNOPQRSTUVWXYZ]",dat$pop, perl=TRUE), "modern", "ancient")  #assumes pops are CAP

tmp <- dat[order(dat$pop),] #Sort by population, subsample 20 from each
d <- by(tmp, tmp["pop"],head,n=20)
reduced_dat<-Reduce(rbind, d)


pca <-ggplot(data=dat, aes(x=PC1,y=PC2, color=ancients, label=pop))  + geom_text(size=2,alpha=0.8) + theme_bw()  + scale_colour_manual(values=c("coral",  "darkgrey"))  + theme(legend.position="none") #colours ancients, greys moderns

pca
```
OR
```
pca <-ggplot(data=dat, aes(x=PC1,y=PC2, color=pop, label=pop))  + geom_text(size=2) + scale_shape_manual(values=rep(c(0:10,12:25,48:57,65:122),18)) + theme_bw()  + scale_colour_manual(values=rep(plot_colours,10))  + theme(legend.position="none")

pca
```
