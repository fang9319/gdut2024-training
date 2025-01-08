## obitools 4 处理测序数据
### 看一下生数据并进行质控, 运行一下代码你会得到两个html文件
```bash
#gzip -cd *.gz | head
head sample1.fq
fastqc sample1.fq
fastqc sample2.fq
```
### 拷贝html文件到自己的电脑
```bahs
scp -P 8080 dell@10.24.22.176:/media/dell/eDNA2/db-training/yourfolder/sample1_fastqc.html \
 /path/yo/your/directory/

scp -P 8080 dell@10.24.22.176:/media/dell/eDNA2/db-training/yourfolder/sample2_fastqc.html \
 /path/yo/your/directory/
```

### R1、R2端数据合并
```bash
obipairing -F sample1.fq -R sample2.fq > paired.fastq
```
### 看一下合并结果
```bash
seqkit stats paired.fastq
head paired.fastq
obicount paired.fastq
```
### 保留成功合并的数据
```bash
obigrep -p 'annotations.mode != "join"' paired.fastq > ali.fastq
```
### demultiplex拆分数据
```bash
obimultiplex -t plate_map.tsv \
-u unidentified.fastq \
ali.fastq \
> ali.assigned.fastq
```
### 去重复
```bash
obiuniq -m sample \
ali.assigned.fastq \
> ali.assigned.uniq.fasta
```
### 去除多余的header
```bash
obiannotate -k count -k merged_sample \
ali.assigned.uniq.fasta \
> ali.assigned.simple.fasta
```

### 降噪
```bash
obiclean -s sample -r 0.05 -H \
  ali.assigned.simple.fasta \
      > ali.assigned.simple.clean.fasta
```

### 筛选
```bash
obigrep -l 150 ali.assigned.simple.clean.fasta \
    > ali.assigned.simple.clean.c5.l150.fasta
```

### 物种注释
#### vsearch
```bash
vsearch --threads 4 --sintax ali.assigned.simple.clean.l150.fasta --db ../ref/fish_sintax.fasta --sintax_cutoff 0.7 --tabbedout sintax.tsv
```
#### blast
```bash
blastn -task blastn -num_threads 4 -evalue 1000 -word_size 7 -max_target_seqs 500 \
-db ref/fish_blast  -outfmt "6 qseqid sseqid evalue length pident nident score bitscore" \
-out blast.out -query ali.assigned.simple.clean.l150.fasta

printf "qseqid\tsseqidLocal\tevalueLocal\tlengthLocal\tpidentLocal\tnidentLocal\tscoreLocal\tbitscoreLocal\n" \
> headers

cat headers blast.out > blast_result.tsv
rm blast.out
rm headers
awk '{if ($5 >= 90 && $4 >= 100) { print } }' blast_result.tsv > blast_result_sorted.tsv 
```
### 切换一下conda环境
```bash
export PATH="/BioAnalyse/miniconda3/envs/be/bin:$PATH"
conda activate be
```

### 导出OTU表
```bash
Rscript export_tab.R --input ali.assigned.simple.clean.l150.fasta --output tab.csv
```
### 下游分析第一步
```bash
Rscript ../make-OTU-obi3.R --sintax sintax.tsv --blast blast_result_sorted.tsv --taxonomy ../ref/reference-library-master.txt --otus tab.csv --output obi_full_OTUs.csv
```