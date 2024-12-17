## obitools 4 处理测序数据
### 看一下生数据并进行质控
```bash
#gzip -cd *.gz | head
head sample1.fastq

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

### 命名匹配
```bash



