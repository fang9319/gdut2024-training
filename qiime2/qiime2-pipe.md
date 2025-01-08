### 准备 metadata.tsv
### 看看导入格式
```bash
qiime tools list-formats --importable
```

### 导入生数据import raw sequences
```bash
qiime tools import \
  --type MultiplexedPairedEndBarcodeInSequence \
  --input-path raw-data/ \
  --output-path temp/multiplexed-seqs.qza
```

### 数据拆分demultiplex；--p-no-anchor-forward-barcode是否从第一个序列开始切割，--p-mixed-orientation T-A建库需加
```bash
qiime cutadapt demux-paired \
  --p-cores 4 \
  --i-seqs temp/multiplexed-seqs.qza \
  --m-forward-barcodes-file metadata.tsv \
  --m-forward-barcodes-column forwardBarcodes \
  --m-reverse-barcodes-file metadata.tsv \
  --m-reverse-barcodes-column reverseBarcodes \
  --p-no-anchor-forward-barcode \
  --p-no-anchor-reverse-barcode \
  --p-error-rate 0 \
  --p-mixed-orientation \
  --o-per-sample-sequences temp/demultiplexed-seqs.qza \
  --o-untrimmed-sequences temp/unassigned.qza \
  --verbose
```

### 切掉前后引物
```bash
qiime cutadapt trim-paired \
  --p-cores 4 \
  --i-demultiplexed-sequences temp/demultiplexed-seqs.qza \
  --p-anywhere-f AAACTCGTGCCAGCCACC \
  --p-anywhere-r GGGTATCTAATCCCAGTTTG \
  --p-error-rate 0.1 \
  --o-trimmed-sequences temp/trimmed-seqs.qza \
  --p-minimum-length 105 \
  --p-quality-cutoff-5end 30 \
  --p-quality-cutoff-3end 30 \
  --verbose
```

### 输出拆分结果，看看进程
```bash
qiime demux summarize \
  --i-data multiplexed-seqs.qza \
  --o-visualization mux.qzv \

qiime tools view temp/mux.qzv


qiime demux summarize \
  --i-data temp/demultiplexed-seqs.qza \
  --o-visualization temp/demultiplexed-seqs.qzv

qiime tools view temp/demultiplexed-seqs.qzv

qiime demux summarize \
  --i-data temp/trimmed-seqs.qza \
  --o-visualization temp/trimmed-seqs.qzv

qiime tools view temp/trimmed-seqs.qzv

```

### dada2降噪
```bash
qiime dada2 denoise-paired \
    --i-demultiplexed-seqs temp/trimmed-seqs.qza \
    --p-trim-left-f 0 --p-trunc-len-f 140 \
    --p-trim-left-r 0 --p-trunc-len-r 140 \
    --p-n-reads-learn 100000 \
    --p-max-ee-f 2.0 \
    --p-max-ee-r 2.0 \
    --p-min-overlap 20 \
    --o-representative-sequences temp/representative-sequences.qza \
    --o-table temp/table.qza \
    --o-denoising-stats temp/denoising-stats.qza
```

### 到处处理过程的统计结果
```bash
qiime tools export \
  --input-path temp/denoising-stats.qza \
  --output-path stats.tsv
```

### 得到ASVs表
```bash
qiime feature-table summarize\
    --i-table temp/table.qza\
    --o-visualization temp/table.qzv\
    --m-sample-metadata-file metadata.tsv

qiime tools view temp/table.qzv
```
### 导出ASVs表
```bash
qiime tools extract --input-path temp/table.qza --output-path results/

biom convert \
    -i results/67d8641b-2a9c-4195-b0f3-c810ac77b9aa/data/feature-table.biom \
    -o results/feature-table.tsv \
    --to-tsv
```

### 导出fasta文件
```bash
qiime feature-table tabulate-seqs \
  --i-data temp/representative-sequences.qza \
  --o-visualization temp/asv.qzv

qiime tools export --input-path temp/asv.qzv --output-path results/

qiime tools view temp/asv.qzv
```

### 用sintax和blast分别进行物种识别
```bash
vsearch --threads 4 --sintax results/sequences.fasta --db ../ref/fish_sintax.fasta --sintax_cutoff 0.7 --tabbedout sintax-q2.tsv

blastn -task blastn -num_threads 4 -evalue 1000 -word_size 7 -max_target_seqs 500 -db ../ref/fish_blast -outfmt "6 qseqid sseqid evalue length pident nident score bitscore" -out results/blast.out -query results/sequences.fasta

printf "qseqid\tsseqidLocal\tevalueLocal\tlengthLocal\tpidentLocal\tnidentLocal\tscoreLocal\tbitscoreLocal\n" > results/headers

cat results/headers results/blast.out > results/blast_result.tsv
rm results/headers
rm results/blast.out
awk '{if ($4 >= 100 && $5 >= 90) { print } }' results/blast_result.tsv > results/blast_result_sorted.tsv 
```
### 切换一下conda环境
```bash
export PATH="/BioAnalyse/miniconda3/envs/be/bin:$PATH"
conda activate be
```

### 下游分析第一步
```bash
Rscript ../make-OTU-qiime2.R --sintax results/sintax.tsv --blast results/blast_result_sorted.tsv --taxonomy ../ref/reference-library-master.txt --otus results/tab.csv --output results/q2_full_OTUs.csv
```



