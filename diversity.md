## Beta Diversity
```
library(vegan)
library(ggplot2)
library(plyr)

#read in metadata
meta<-read.delim("bird_meta.txt", header=T)

#16S, dropped 4 Samples (n=56 total)
s16<-read.delim("16S_table_tax_filt.txt", header=T, row.names=1)
s16<-s16[,-52]
s16<-t(s16)

#remove non-captive birds
#s16<-s16[c(-2,-4,-8,-10,-15,-16,-20,-21,-33,-35),]

#calculate PCoA based on Bray-curtis and calculate the percent variation explained
set.seed(503)
#s16<-rrarefy(s16, sample=330)
s16.pcoa<-capscale(s16 ~ 1, distance='bray')
100*round(s16.pcoa$CA$eig[1]/sum(s16.pcoa$CA$eig), 3)
#25.8
100*round(s16.pcoa$CA$eig[2]/sum(s16.pcoa$CA$eig), 3)
#12.9

#extract the coordinates
s16.scores<-scores(s16.pcoa)
s16.coords<-as.data.frame(s16.scores$sites)
s16.coords$SampleID<-rownames(s16.coords)
s16.coords<-merge(s16.coords, meta, by=c('SampleID'))

#plot it yo
ggplot(s16.coords, aes(MDS1, MDS2, colour=Treatment))+  
  geom_point(aes(size=2))+
  theme_bw()+
  xlab("PC1-26.3%")+
  ylab("PC2-13.7%")+
  theme(text = element_text(size=14),
        axis.text = element_text(size=14), legend.text=element_text(size=14))
```

## Adonis
```
#read in metadata
meta<-read.delim("bird_meta.txt", header=T)

#16S, dropped 4 Samples (n=56 total)
s16<-read.delim("16S_table_tax_filt.txt", header=T, row.names=1)
s16<-s16[,-52]
s16<-t(s16)

s16.dis<-as.data.frame(as.matrix(vegdist(s16, method='bray')))
s16.dis2<-s16.dis
s16.dis2$SampleID<-row.names(s16.dis2)
s16.dis2<-merge(s16.dis2, meta, by=c('SampleID'))
s16.dis3<-s16.dis2[,2:52]
adonis(s16.dis3 ~ Treatment, data=s16.dis2, permutations = 10000)
#p=9.9e-5, F(3,47)=47.82, R2=0.75
```
## Alpha Diversity
#rarefy data
#rowSums(s16)
s16<-rrarefy(s16, sample=333)

#Shannon
s16.shan<-diversity(s16, index='shannon')

#OTUs observed
s16.otus<-rowSums(s16>0)

#Pielou's Evenness
s16.even<-(diversity(s16))/s16.otus

#catenate data into a single frame
s16.div<-as.data.frame(cbind(s16.shan, s16.otus, s16.even))

#add sample IDs
s16.div$SampleID<-row.names(s16.div)

#fix coloumn names
names(s16.div)<-c('Shannon', 'OTUs_Obs', 'Pielous_Even', 'SampleID')

#read in metadata
meta<-read.delim("bird_meta.txt", header=T)

#add metadata
s16.div<-merge(s16.div, meta, by='SampleID')

otus<-ggplot(s16.div, aes(Treatment, OTUs_Obs))+
  geom_boxplot()+
  theme_bw()

shan<-ggplot(s16.div, aes(Treatment, Shannon))+
  geom_boxplot()+
  theme_bw()

even<-ggplot(s16.div, aes(Treatment, Pielous_Even))+
  geom_boxplot()+
  theme_bw()

library(gridExtra)
grid.arrange(otus, shan, even, ncol=2)
```

## Test signficane of alpha diversity
```
#t-test for significance
pairwise.t.test(s16.div$Shannon, s16.div$Treatment, p.adjust.method = 'hochberg')
pairwise.t.test(s16.div$OTUs_Obs, s16.div$Treatment, p.adjust.method = 'hochberg')
pairwise.t.test(s16.div$Pielous_Even, s16.div$Treatment, p.adjust.method = 'hochberg')
```
