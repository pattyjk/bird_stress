  ## Distance boxplots
  ```
  source activate qiime1
  #calculate bray-curtis similarity
  beta_diversity.py -i 16S_table_tax_filt.biom -m bray_curtis -o beta_div
  
  #make boxplots
  make_distance_boxplots.py -m /home/pattyjk/Bird_Stress_Project/Reads/16S/16S_map.txt -o distance_plots -f Treatment --save_raw_data -d beta_div/bray_curtis_16S_table_tax_filt.txt
  ```
