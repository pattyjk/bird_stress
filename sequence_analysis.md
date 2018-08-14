## Demultiplex reads
```
python prep_fastq_for_uparse_paired.py -o 16S_demult -m 16S_map.txt -i Madden_16s_NoIndex_L001_R1_001.fastq -r Madden_16s_NoIndex_L001_R3_001.fastq -b Madden_16s_NoIndex_L001_R2_001.fastq -c

cp 16S_demult/demultiplexed_seqs_* /home/pattyjk/Bird_Stress_Project/Reads/mergedfastq/
```

## fix read headers
```
cd ..
cd mergedfastq

#fix bacteria
sed -i 's/barcodelabel/sample/g' demultiplexed_seqs_1.fq
sed -i 's/barcodelabel/sample/g' demultiplexed_seqs_2.fq
sed -i 's/ 1:N:0:/1:N:0/g' demultiplexed_seqs_1.fq
sed -i 's/ 3:N:0:/3:N:0/g' demultiplexed_seqs_2.fq

cd ..
```

## use ITSx to remove ribosomal fragments from ITS reads
```
#add code
```

## Join paired ends
```
#usearch v 10.0.24

#fungi
./usearch64 -fastq_mergepairs mergedfastq/demultiplexed_seqs_1.fq -reverse mergedfastq/demultiplexed_seqs_2.fq -fastqout mergedfastq/ITS_merged.fq -fastq_maxdiffs 10 -fastq_merge_maxee 1.0
##only 24% of fungal reads could be merged will do the anaysis with both merged reads and R2

#bacteria
./usearch64 -fastq_mergepairs mergedfastq/demultiplexed_seqs_1.fq -reverse mergedfastq/demultiplexed_seqs_2.fq -fastqout mergedfastq/16S_merged.fq -fastq_maxdiffs 10 -fastq_merge_maxee 1.0
##88% could be joined, normal amount
```

## Dereplicate sequences
```
#ITS
./usearch64 -fastx_uniques mergedfastq/demultiplexed_seqs_2a.fq -fastqout mergedfastq/fungiR2_uniques_combined_merged.fastq -sizeout
./usearch64 -fastx_uniques mergedfastq/ITS_merged.fq -fastqout mergedfastq/fungi_uniques_combined_merged.fastq -sizeout

#16S
./usearch64 -fastx_uniques mergedfastq/16S_merged.fq -fastqout mergedfastq/16S_uniques_combined_merged.fastq -sizeout
```

## Remove Singletons
```
#16S
./usearch64 -sortbysize mergedfastq/16S_uniques_combined_merged.fastq -fastqout mergedfastq/16S_nosigs_uniques_combined_merged.fastq -minsize 2

#ITS
./usearch64 -sortbysize mergedfastq/fungiR2_uniques_combined_merged.fastq -fastqout mergedfastq/fungiR2_nosigs_uniques_combined_merged.fastq -minsize 2
./usearch64 -sortbysize mergedfastq/fungi_uniques_combined_merged.fastq -fastqout mergedfastq/fungi_nosigs_uniques_combined_merged.fastq -minsize 2
```

## Precluster sequences
```
#bacteria
./usearch64 -cluster_fast mergedfastq/16S_nosigs_uniques_combined_merged.fastq -centroids_fastq mergedfastq/16S_denoised_nosigs_uniques_combined_merged.fastq -id 0.9 -maxdiffs 5 -abskew 10 -sizein -sizeout -sort size

#fungi
./usearch64 -cluster_fast mergedfastq/fungi_nosigs_uniques_combined_merged.fastq -centroids_fastq mergedfastq/fungi_denoised_nosigs_uniques_combined_merged.fastq -id 0.9 -maxdiffs 5 -abskew 10 -sizein -sizeout -sort size
./usearch64 -cluster_fast mergedfastq/fungiR2_nosigs_uniques_combined_merged.fastq -centroids_fastq mergedfastq/fungiR2_denoised_nosigs_uniques_combined_merged.fastq -id 0.9 -maxdiffs 5 -abskew 10 -sizein -sizeout -sort size
```

## Closed reference pick against Silva and UNITE
```
#bacteria
./usearch64 -usearch_global mergedfastq/16S_denoised_nosigs_uniques_combined_merged.fastq -id 0.97 -db silva_132_97_16S.fna  -strand plus -uc mergedfastq/16S_ref_seqs.uc -dbmatched mergedfastq/16S_closed_reference.fasta -notmatchedfq mergedfastq/16S_failed_closed.fq

#get unite
#wget https://files.plutof.ut.ee/doi/27/19/271988E02F4673D62D4E43404921596FA73671F2E00507E0727884BE9082C513.zip
#unzip 271988E02F4673D62D4E43404921596FA73671F2E00507E0727884BE9082C513.zip

#fungi
./usearch64 -usearch_global mergedfastq/fungi_denoised_nosigs_uniques_combined_merged.fastq -id 0.97 -db unite97.fna -strand plus -uc mergedfastq/fungi_ref_seqs.uc -dbmatched mergedfastq/fungi_closed_reference.fasta -notmatchedfq mergedfastq/fungi_failed_closed.fq
./usearch64 -usearch_global mergedfastq/fungiR2_denoised_nosigs_uniques_combined_merged.fastq -id 0.97 -db unite97.fna -strand plus -uc mergedfastq/fungiR2_ref_seqs.uc -dbmatched mergedfastq/fungiR2_closed_reference.fasta -notmatchedfq mergedfastq/fungiR2_failed_closed.fq
```

## Sort by size and de novo pick
```
#bacteria
./usearch64 -sortbysize mergedfastq/16S_failed_closed.fq -fastaout mergedfastq/16S_sorted_failed_closed.fq
./usearch64 -cluster_otus mergedfastq/16S_sorted_failed_closed.fq -minsize 2 -otus mergedfastq/16S_denovo_otus.fasta -relabel OTU_dn_ -uparseout mergedfastq/16S_denovo_out.up

#fungi
./usearch64 -sortbysize mergedfastq/fungi_failed_closed.fq -fastaout mergedfastq/fungi_sorted_failed_closed.fq
./usearch64 -cluster_otus mergedfastq/fungi_sorted_failed_closed.fq -minsize 2 -otus mergedfastq/fungi_denovo_otus.fasta -relabel fOTU_dn_ -uparseout mergedfastq/fungi_denovo_out.up

./usearch64 -sortbysize mergedfastq/fungiR2_failed_closed.fq -fastaout mergedfastq/fungiR2_sorted_failed_closed.fq
./usearch64 -cluster_otus mergedfastq/fungiR2_sorted_failed_closed.fq -minsize 2 -otus mergedfastq/fungiR2_denovo_otus.fasta -relabel fOTU_dn_ -uparseout mergedfastq/fungiR2_denovo_out.up
```

## Combine the rep sets between de novo and reference-based OTU picking rep sets

``` 
#bacteria
cat mergedfastq/16S_closed_reference.fasta mergedfastq/16S_denovo_otus.fasta > mergedfastq/16S_full_rep_set.fna

#fungi
cat mergedfastq/fungi_closed_reference.fasta mergedfastq/fungi_denovo_otus.fasta > mergedfastq/fungi_full_rep_set.fna
cat mergedfastq/fungiR2_closed_reference.fasta mergedfastq/fungiR2_denovo_otus.fasta > mergedfastq/fungiR2_full_rep_set.fna
```

## Map rep_set back to pre-dereplicated sequences and make OTU tables
```
#bacteria
./usearch64 -usearch_global mergedfastq/16S_merged.fq -db mergedfastq/16S_full_rep_set.fna  -strand plus -id 0.97 -uc mergedfastq/16S_OTU_map.uc -otutabout mergedfastq/16S_OTU_table.txt -biomout mergedfastq/16S_OTU_jsn.biom

#fungi
./usearch64 -usearch_global mergedfastq/ITS_merged.fq -db mergedfastq/fungi_full_rep_set.fna  -strand plus -id 0.97 -uc mergedfastq/fungi_OTU_map.uc -otutabout mergedfastq/ITS_OTU_table.txt -biomout mergedfastq/ITS_OTU_jsn.biom

./usearch64 -usearch_global mergedfastq/demultiplexed_seqs_2a.fq -db mergedfastq/fungiR2_full_rep_set.fna  -strand plus -id 0.97 -uc mergedfastq/fungiR2_OTU_map.uc -otutabout mergedfastq/ITSR2_OTU_table.txt -biomout mergedfastq/ITSR2_OTU_jsn.biom
```

## Assign taxonomy to the OTUs in QIIME (v. 1.9.1)
```
source activate qiime1
#bacteria
assign_taxonomy.py -i mergedfastq/16S_full_rep_set.fna -o mergedfastq/16S_taxonomy -t '/home/pattyjk/SILVA_132_QIIME_release/taxonomy/16S_only/97/consensus_taxonomy_7_levels.txt' -r '/home/pattyjk/SILVA_132_QIIME_release/rep_set/rep_set_16S_only/97/silva_132_97_16S.fna'

#fungi
assign_taxonomy.py -i mergedfastq/fungi_full_rep_set.fna -o mergedfastq/fungi_taxonomy -r '/home/pattyjk/UNITE/sh_refs_qiime_ver7_97_s_01.12.2017.fasta' -t '/home/pattyjk/UNITE/sh_taxonomy_qiime_ver7_97_s_01.12.2017.txt' 

assign_taxonomy.py -i mergedfastq/fungiR2_full_rep_set.fna -o mergedfastq/fungiR2_taxonomy -r '/home/pattyjk/UNITE/sh_refs_qiime_ver7_97_s_01.12.2017.fasta' -t '/home/pattyjk/UNITE/sh_taxonomy_qiime_ver7_97_s_01.12.2017.txt' 
```

## Add taxonomy to OTU table
```
#bacteria
biom convert -i mergedfastq/16S_OTU_table.txt -o mergedfastq/16S_OTU_table.biom --to-hdf5 --table-type='OTU table'

biom add-metadata -i mergedfastq/16S_OTU_table.biom -o mergedfastq/16S_table_tax.biom --observation-metadata-fp=mergedfastq/16S_taxonomy/16S_full_rep_set_tax_assignments.txt --sc-separated=taxonomy --observation-header=OTUID,taxonomy

#fungi
biom convert -i mergedfastq/ITS_OTU_table.txt -o mergedfastq/ITS_OTU_table.biom --to-hdf5 --table-type='OTU table'

biom add-metadata -i mergedfastq/ITS_OTU_table.biom -o mergedfastq/ITS_table_tax.biom --observation-metadata-fp=mergedfastq/fungi_taxonomy/fungi_full_rep_set_tax_assignments.txt --sc-separated=taxonomy --observation-header=OTUID,taxonomy

biom convert -i mergedfastq/ITSR2_OTU_table.txt -o mergedfastq/ITSR2_OTU_table.biom --to-hdf5 --table-type='OTU table'

biom add-metadata -i mergedfastq/ITSR2_OTU_table.biom -o mergedfastq/ITSR2_table_tax.biom --observation-metadata-fp=mergedfastq/fungiR2_taxonomy/fungiR2_full_rep_set_tax_assignments.txt --sc-separated=taxonomy --observation-header=OTUID,taxonomy
```

## Filter out unwanted taxa Chloroplasts, mitochondria, Archaea
```
#filter unwanted bacterial taxa
filter_taxa_from_otu_table.py -i  mergedfastq/16S_table_tax.biom -o mergedfastq/16S_table_tax_filt.biom -n D_4__Mitochondria,D_3__Chloroplast,D_0__Archaea

#summarize taxonomy
summarize_taxa.py -i mergedfastq/16S_table_tax_filt.biom -o mergedfastq/16S_taxa_sum
summarize_taxa.py -i mergedfastq/ITS_table_tax.biom -o ITS_taxa_sum
summarize_taxa.py -i mergedfastq/ITSR2_table_tax.biom -o ITSR2_taxa_sum
```

## Convert filtered OTU tables to text files
```
biom convert -i mergedfastq/16S_table_tax_filt.biom -o mergedfastq/16S_table_tax_filt.txt --table-type='OTU table' --header-key=taxonomy --to-tsv

biom convert -i mergedfastq/ITS_table_tax.biom -o mergedfastq/ITS_table_tax_filt.txt --table-type='OTU table' --header-key=taxonomy --to-tsv

biom convert -i mergedfastq/ITSR2_table_tax.biom -o mergedfastq/ITSR2_table_tax_filt.txt --table-type='OTU table' --header-key=taxonomy --to-tsv
```


