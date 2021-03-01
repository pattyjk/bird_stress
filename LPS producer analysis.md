## LPS producer analysis
```
# read in taxonomic data
bird_fams<-read.delim('16S_table_tax_filt_L5.txt', header=T)

#read in LPS data
lps<-read.delim('L5_WithLPSProducers (1).csv', header=T, sep=',')

#merge dataframes
fams_lps<-merge(bird_fams, lps, by='X')

#read in metadata
meta<-read.delim("bird_meta.txt", header=T)

# melt data
library(reshape)
fams_lps_m<-melt(fams_lps)

#tack on meta data
fams_lps2<-merge(fams_lps_m, meta, by.x='variable', by.y='SampleID')

#plot 
library(ggplot2)
ggplot(fams_lps2,  aes(Probable.LPS.producer, value, fill=Treatment))+
geom_boxplot()+
xlab("LPS produce status")+
ylab("Relative abundance")+
theme_bw()

#make column that is aggregate between treatment/LPS status
fams_lps2$new<-paste(fams_lps2$Probable.LPS.producer, fams_lps2$Treatment, sep=',')


#Welch's t-test between categories
pairwise.t.test(fams_lps2$value, fams_lps2$new, p.adjust.method = 'hochberg')

# get means for each category
library(plyr)


sum<-ddply(fams_lps2, c("Treatment", "Probable.LPS.producer"), summarize, mean=mean(value), sd=sd(value), n=length(value), se=sd/n)
write.table(sum, file='lps_summary.txt', quote=F)

```
