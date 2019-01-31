## Correlations between diversity and meta data
### Alpha diversity
```
#read in metadata
meta<-read.delim("bird_meta.txt", header=T)

#subsample map to only include experimental birds
meta_exp<-subset(meta, meta$CaptiveorNot == 'Captive')

#16S, dropped 4 Samples (n=56 total)
s16<-read.delim("16S_table_tax_filt.txt", header=T, row.names=1)
s16<-s16[,-52]
s16<-t(s16)

dim(s16)
51 987

#filter out low abundance samples
s16<-s16[c(-45:-51),]

dim(s16)
44 987

#select only exp birds in OTU table
s16_exp<-s16[row.names(s16) %in% meta_exp$SampleID,]
dim(s16_exp)
33 by 987

#rarefy data
library(vegan)
#rowSums(s16)
s16_exp<-rrarefy(s16_exp, sample=333)

#Shannon
s16.shan<-diversity(s16_exp, index='shannon')

#OTUs observed
s16.otus<-rowSums(s16_exp>0)

#Pielou's Evenness
s16.even<-(diversity(s16_exp))/s16.otus

#catenate data into a single frame
s16.div<-as.data.frame(cbind(s16.shan, s16.otus, s16.even))

#add sample IDs
s16.div$SampleID<-row.names(s16.div)

#fix coloumn names
names(s16.div)<-c('Shannon', 'OTUs_Obs', 'Pielous_Even', 'SampleID')

#add metadata
s16.div<-merge(s16.div, meta_exp, by='SampleID')
s16.div<-s16.div[-30,]

#test correlations between diversity and parameters
cor.test(s16.div$DaysTotalCaptivity, s16.div$Shannon, method='spearman')
cor.test(s16.div$RelativeSpleenWt, s16.div$Shannon, method='spearman')
cor.test(s16.div$WeightChange, s16.div$Shannon, method='spearman')
cor.test(s16.div$AcuteCORT2, s16.div$Shannon, method='spearman')
cor.test(s16.div$BaseCORT2, s16.div$Shannon, method='spearman')

#test between sexes
t.test(s16.div$Shannon ~ s16.div$Sex)
#p=0.7

#nothing is significant, unsuprisngly
```

### Beta diversity
```
#calculate a PCoA
s16.pcoa<-capscale(s16_exp ~ 1, distance='bray')

#extract the coordinates
s16.scores<-scores(s16.pcoa)
s16.coords<-as.data.frame(s16.scores$sites)
s16.coords$SampleID<-rownames(s16.coords)
s16.coords<-merge(s16.coords, meta, by=c('SampleID'))
s16.coords<-s16.coords[-30,]

ggplot(s16.coords, aes(s16.coords$WeightChange, MDS1, colour=Treatment))+
  geom_point()+
  theme_bw()

cor.test(s16.coords$MDS1, s16.coords$DaysCaptivity, method='spearman')
#not sig
cor.test(s16.coords$MDS1, s16.coords$DaysTotalCaptivity, method='spearman')
#not sig
cor.test(s16.coords$MDS1, s16.coords$BaselineCORT, method='spearman')
#not sig

cor.test(s16.coords$MDS2, s16.coords$RelativeSpleenWt, method='spearman')
#p=0.002, Rho=0.62
cor.test(s16.coords$MDS1, s16.coords$AcuteCORT2, method='spearman')
#p=0.041, Rho=-0.34
cor.test(s16.coords$MDS1, s16.coords$FractionWeightChange, method='spearman')
#p=0.01, rho=-0.51
```
