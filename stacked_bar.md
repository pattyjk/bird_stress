## Stacked barplot of the rel. abun. of bacterial families
```
#read family-level data
bird_fams<-read.delim('16S_table_tax_filt_L5.txt', header=T, row.names=1)
dim(bird_fams)
#160,51

#remove samples that have low read counts
bad<-c('X25', 'X23', 'X8', 'X11', 'X28', 'X24')
bird_fams<-bird_fams[,-grep(paste(bad, collapse='|'), names(bird_fams))]
dim(bird_fams)
#160, 45
bird_fams<-bird_fams[,-45]
dim(bird_fams)
#160, 44

#get row sums
bird_fams$sum<-rowSums(bird_fams)
bird_fams<-bird_fams[order(bird_fams$sum, decreasing=T) , ]

#filter out low abundance taxa
bird_fams<-bird_fams[1:20,]
#dim(bird_fams)

#remove sum column
bird_fams<-as.data.frame(bird_fams[,-grep('sum', names(bird_fams))])

#get 'others' category (things not in top20)
others<-1-colSums(bird_fams)
bird_fams<-rbind(bird_fams, others)
rownames(bird_fams)[21]<-"Others;Others;Others;Others;Others;Others;Others"

#add taxonomy back
bird_fams$taxonomy<-row.names(bird_fams)

#melt data
library(reshape)
fam_m<-melt(bird_fams)
names(fam_m)<-c("Taxonomy", "SampleID", "Rel_abun")

#change to percent
fam_m$Rel_abun<-fam_m$Rel_abun*100

#split taxonomy
library(tidyr)
library(stringi)
fam_m_split<-separate(fam_m, Taxonomy, sep=";", into=c("Kingdom", "Phylum", "Class", "Order", "Family", "Genus", "Species"))
fam_m_split$Family<-gsub("D_4__", "", fam_m_split$Family)

#read in metadata
meta<-read.delim("bird_meta.txt", header=T)

#bind metadata
fam_m_split<-merge(fam_m_split, meta, by='SampleID')

#fix names of treatments
fam_m_split$Treatment<-gsub('Experimental', 'Stressed', fam_m_split$Treatment)
fam_m_split$Treatment<-gsub('No Tx', 'Wild Birds', fam_m_split$Treatment)

bird_fams<-bird_fams[order(bird_fams$sum, decreasing=T) , ]

#reorder samples
#fam_m_split<-fam_m_split[order(fam_m_split$SampleID, decreasing=T),]

#read in the best pallette
pal<-c("#771155", "#CC99BB", "#114477", "#4477AA", "#117777", "#44AAAA", "#77CCCC", "#117744", "#44AA77", "#88CCAA", "#777711", "#AAAA44", "#DDDD77", "#774411", "#AA7744", "#DDAA77", "#771122", "#AA4455", "#DD7788","#41AB5D", "#252525", "#525252", "#737373", "#969696")

#plot data as stacked barplot
library(ggplot2)
ggplot(fam_m_split, aes(SampleID, Rel_abun, fill=Family))+
  geom_bar(stat='identity')+
  scale_y_continuous(expand=c(0,0))+
  scale_fill_manual(values=pal)+
  guides(fill=guide_legend(ncol=1))+
  xlab("")+
  ylab("Relative Abundance")+
  theme_bw()+
  facet_wrap(~Treatment, scales = 'free')+
  theme(text = element_text(size=14), axis.text.x = element_blank())
```

## Plot Phylum-level
```
ggplot(fam_m_split, aes(SampleID, Rel_abun, fill=Phylum))+
  geom_bar(stat='identity')+
  scale_y_continuous(expand=c(0,0))+
  scale_fill_manual(values=pal)+
  guides(fill=guide_legend(ncol=1))+
  xlab("")+
  ylab("Relative Abundance")+
  theme_bw()+
  facet_wrap(~Treatment, scales = 'free')+
  theme(text = element_text(size=14), axis.text.x = element_blank())
```
