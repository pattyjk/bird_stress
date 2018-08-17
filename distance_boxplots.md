  ## Distance boxplots
  ```
  source activate qiime1
  #QIIME 1.9.1
  #calculate bray-curtis similarity
  beta_diversity.py -i 16S_table_tax_filt.biom -m bray_curtis -o beta_div
  
  #make boxplots
  make_distance_boxplots.py -m /home/pattyjk/Bird_Stress_Project/Reads/16S/16S_map.txt -o distance_plots -f Treatment --save_raw_data -d beta_div/bray_curtis_16S_table_tax_filt.txt
  ```
  
## Plot data in R
```
#R v. 3.4
#read data
treat_dist<-read.delim('beta_distance.txt', header=T)

#pull out categories of interest
treat_dist<-treat_dist[,3:12]

#melt the data
library(reshape)
treat_dist_m<-melt(treat_dist)
str(treat_dist_m)

#change to similarity
treat_dist_m$value<-1-treat_dist_m$value
#plot data
library(ggplot2)
ggplot(treat_dist_m, aes(variable, value))+
  geom_boxplot()+
  theme_bw()+
  coord_flip()+
  xlab("")+
  ylab("Bray-Curtis Similarity")
```
## Test for significance
```
#Welch's t-test between categories
pairwise.t.test(treat_dist_m$value, treat_dist_m$variable, p.adjust.method = 'hochberg')
```
