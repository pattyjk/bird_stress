## Microbial abundance
```
meta<-read.delim("bird_meta.txt", header=T)
meta<-subset(meta, Treatment == 'Control' | Treatment == 'Experimental' | Treatment == 'Recovery')
library(ggplot2)
tsa<-ggplot(meta, aes(Treatment, meta$TotalCFUTSA))+
  geom_boxplot()+
  scale_y_log10()+
  theme_bw()+
  ylab("Log10 CFU- TSA")+
  xlab("")
pairwise.t.test(meta$TotalCFUTSA, meta$Treatment, p.adjust.method = 'hochberg')

pda<-ggplot(meta, aes(Treatment, meta$TotalCFUPDA))+
  geom_boxplot()+
  scale_y_log10()+
  theme_bw()+
  ylab("Log10 CFU- PDA")+
  xlab("")
pairwise.t.test(meta$TotalCFUPDA, meta$Treatment, p.adjust.method = 'hochberg')

mac<-ggplot(meta, aes(Treatment, meta$TotalCFUMAC))+
  geom_boxplot()+
  scale_y_log10()+
  theme_bw()+
  ylab("Log10 CFU- Mackonkey")+
  xlab("")
pairwise.t.test(meta$TotalCFUMAC, meta$Treatment, p.adjust.method = 'hochberg')

myco<-ggplot(meta, aes(Treatment, meta$TotalCFUMYCO))+
  geom_boxplot()+
  scale_y_log10()+
  theme_bw()+
  ylab("Log10 CFU- Myco")+
  xlab("")
pairwise.t.test(meta$TotalCFUMYCO, meta$Treatment, p.adjust.method = 'hochberg')

ba<-ggplot(meta, aes(Treatment, meta$TotalCFUBA))+
  geom_boxplot()+
  scale_y_log10()+
  theme_bw()+
  ylab("Log10 CFU- BA")+
  xlab("")

library(gridExtra)
grid.arrange(tsa, mac, pda, myco, ba)
```

## Test significance
```
pairwise.t.test(meta$TotalCFUBA, meta$Treatment, p.adjust.method = 'hochberg')
```
