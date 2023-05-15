## RIBBITR 16S sequence processing SOP
```
#load QIIME 2022.8
source activate qiime2-2022.2

#load raw FASTQ reads into QIIME
#load more as more runs are added
qiime tools import --type EMPSingleEndSequences --input-path ./reads --output-path ribbitr_seqs.qza

#demultiplex reads
qiime demux emp-single \
  --i-seqs ribbitr_seqs.qza \
  --m-barcodes-file ribbitr_16s_map.txt \
  --m-barcodes-column BarcodeSequence \
  --o-per-sample-sequences ribbitr_demux.qza \
  --o-error-correction-details ribbitr_demux-details.qza \
  --p-no-golay-error-correction
  
#quality filer
qiime quality-filter q-score \
 --i-demux ribbitr_demux.qza \
 --o-filtered-sequences ribbitr_demux-filtered.qza \
 --o-filter-stats ribbitr_demux-filter-stats.qza
 
  #call ASVs with deblur
  qiime deblur denoise-16S \
  --i-demultiplexed-seqs ribbitr_demux-filtered.qza \
  --p-trim-length 120 \
  --o-representative-sequences ribbitr_rep-seqs-deblur.qza \
  --o-table ribbitr_table-deblur.qza \
  --p-sample-stats \
  --o-stats ribbitr_deblur-stats.qza
 
 #make phylogenetic tree with fasttree
 qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences ribbitr_rep-seqs-deblur.qza \
  --output-dir phylogeny-align-to-tree-mafft-fasttree

#assign taxonomy to  SILVA with sklearn
qiime feature-classifier classify-sklearn   --i-classifier silva-138-99-515-806-nb-classifier.qza   --i-reads  ribbitr_rep-seqs-deblur.qza   --o-classification ribbitr_taxonomy.qza

qiime metadata tabulate \
  --m-input-file ribbitr_taxonomy.qza \
  --o-visualization taxonomy.qzv
 
# convert ASV table to biom/text
 qiime tools export --input-path ribbitr_table-deblur.qza --output-path asv_table
 biom convert -i asv_table/feature-table.biom -o asv_table.txt --to-tsv
 
 #export taxonomy file
 qiime tools export --input-path taxonomy.qzv --output-path taxonomy
 ```
