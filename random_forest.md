## Random forest of OTUs
```
library("randomForest")
library("plyr")
library("rfUtilities")
library("caret")

#read in mapping file
meta<-read.delim("bird_meta.txt", header=T)

#read in OTU table
s16<-read.delim("16S_table_tax_filt.txt", header=T, row.names=1)
s16<-s16[,-52]
dim(s16)
#987 OTUs by 51 samples


#how many nonzero counts?
otu_nonzero_counts<-apply(s16, 1, function(y) sum(length(which(y > 0))))
hist(otu_nonzero_counts, breaks=100, col="grey", main="", ylab="Number of OTUs", xlab="Number of Non-Zero Values")

#remove rare taxa
remove_rare <- function( table , cutoff_pro ) {
  row2keep <- c()
  cutoff <- ceiling( cutoff_pro * ncol(table) )  
  for ( i in 1:nrow(table) ) {
    row_nonzero <- length( which( table[ i , ]  > 0 ) ) 
    if ( row_nonzero > cutoff ) {
      row2keep <- c( row2keep , i)
    }
  }
  return( table [ row2keep , , drop=F ])
}

otu_table_rare_removed <- remove_rare(table=s16, cutoff_pro=0.1)
dim(otu_table_rare_removed)
#178 OTUs by 51 samples

#scale data
otu_table_scaled <- scale(otu_table_rare_removed, center = TRUE, scale = TRUE)  

#run RF model
otu_table_scaled_treatment <- data.frame(t(otu_table_scaled))
otu_table_scaled_treatment$SampleID<-row.names(otu_table_scaled_treatment)
meta_sub<-as.data.frame(meta[,c(1,11)])
names(meta_sub)<-c("SampleID", "Treatment")
otu_table_scaled_treatment<-merge(otu_table_scaled_treatment, meta_sub, by=c("SampleID"))
otu_table_scaled_treatment<-otu_table_scaled_treatment[,-1]

set.seed(501)
RF_treatment_classify<-randomForest(x=otu_table_scaled_treatment[,1:(ncol(otu_table_scaled_treatment)-1)], y=otu_table_scaled_treatment[ , ncol(otu_table_scaled_treatment)] , ntree=501, importance=TRUE, proximities=TRUE)

#permutation test
RF_treatment_classify_sig<-rf.significance(x=RF_treatment_classify, xdata=otu_table_scaled_treatment[,1:(ncol(otu_table_scaled_treatment)-1)], nperm=1000 , ntree=501)

#identifying important features
RF_state_classify_imp <- as.data.frame(RF_treatment_classify$importance)
RF_state_classify_imp$features <- rownames( RF_state_classify_imp )
RF_state_classify_imp_sorted <- arrange( RF_state_classify_imp  , desc(MeanDecreaseAccuracy)  )
barplot(RF_state_classify_imp_sorted$MeanDecreaseAccuracy, ylab="Mean Decrease in Accuracy (Variable Importance)", main="RF Classification Variable Importance Distribution")

#top 50 features
barplot(RF_state_classify_imp_sorted[1:50,"MeanDecreaseAccuracy"], las=2, names.arg=RF_state_classify_imp_sorted[1:10,"features"] , ylab="Mean Decrease in Accuracy (Variable Importance)", main="Classification RF")  
```

## Plot mean abundance of top 50 most important OTUs
```
top50_feat<-as.data.frame(RF_state_classify_imp_sorted$features[1:20])
names(top50_feat)<-c("OTU")
str(top50_feat)

#read in OTU table, extract taxonomy
s16<-read.delim("16S_table_tax_filt.txt", header=T, row.names=1)
tax<-as.data.frame(s16$taxonomy)
names(tax)<-c("taxonomy")
OTU<-row.names(s16)
s16_tax<-cbind(OTU, tax)
str(s16_tax)
s16<-s16[,-52]

#change to relative abundance
library(vegan)
s16<-decostand(s16, method = 'total')
#check to make sure rel abund
#rowSums(s16[,-52])

#extract top 50 from OTU table
s16.top50<-s16[rownames(s16) %in% top50_feat$OTU,]
dim(s16.top50)  
s16.top50$OTU<-row.names(s16.top50)
names(s16.top50)
rownames(s16.top50)

#get mean relative abundance of each OTU in the 4 cats
library(reshape)
top50_m<-melt(s16.top50)
names(top50_m)<-c("OTU", "SampleID", 'Rel_abund')
library(plyr)
top50_m<-merge(top50_m, meta, by='SampleID')
top50_sum<-ddply(top50_m, c('OTU', "Treatment"), summarize, mean=mean(Rel_abund), sd=sd(Rel_abund), n=length(Rel_abund), se=sd/n)
top50_sum<-merge(top50_sum, s16_tax, by='OTU')

#split taxonomy into groups

library(tidyr)
library(stringi)
top50_sum<-separate(top50_sum, taxonomy, sep=";", into=c("Kingdom", "Phylum", "Class", "Order", "Family", "Genus", "Species"))

#fix the NAs and taxonomy
top50_sum$Genus[is.na(top50_sum$Genus)] <- "Unassigned"
top50_sum$Family[is.na(top50_sum$Genus)] <- "Unassigned"
top50_sum$Order[is.na(top50_sum$Genus)] <- "Unassigned"
top50_sum$Class[is.na(top50_sum$Genus)] <- "Unassigned"
top50_sum$Genus<-gsub('D_5__', '', top50_sum$Genus)
top50_sum$Family<-gsub('D_4__', '', top50_sum$Family)
top50_sum$Order<-gsub('D_3__', '', top50_sum$Order)
top50_sum$Class<-gsub('D_2__', '', top50_sum$Class)
top50_sum$Phylum<-gsub('D_1__', '', top50_sum$Phylum)

#fix treatment
top50_sum$Treatment<-gsub('No Tx', 'Wild Birds', top50_sum$Treatment)
top50_sum$Treatment<-gsub('Experimental', 'Stressed', top50_sum$Treatment)
```

## plot the data
```
colour_tax<-rainbow(41, s=1, v=1)[sample(1:41,41)]
pal<-c("#771155", "#CC99BB", "#114477", "#4477AA", "#117777", "#44AAAA", "#77CCCC", "#117744", "#44AA77", "#88CCAA", "#777711", "#AAAA44", "#DDDD77", "#774411", "#AA7744", "#DDAA77", "#771122", "#AA4455", "#DD7788","#41AB5D", "#252525", "#525252", "#737373", "#969696")

library(ggplot2)
ggplot(top50_sum, aes(OTU, mean, fill=Genus))+
  geom_bar(stat='identity')+
  scale_fill_manual(values=pal)+
  facet_wrap(~Treatment, ncol = 4)+
  theme_bw()+
  coord_flip()+
  ggtitle("Top 20 OTUs in distinguishing categories")+
  guides(fill=guide_legend(ncol=1))+
  ylab("Mean Relative Abundance")+
  xlab("")+
  geom_errorbar(aes(ymax=mean+se, ymin=mean-se, width=0.2), stat="identity")
```
