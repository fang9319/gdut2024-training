# crabs旧版脚本
## ncbi:根据准备好的物种名录，在ncbi上下载序列，一般使用nucleotide数据库.
```bash 
crabs db_download --source ncbi --database nucleotide --query '12S[All Fields] AND ("1"[SLEN] : "50000"[SLEN])' --species elas_parts.csv --output elas_parts.fasta --keep_original yes --email fangliufree@gmail.com --batchsize 50000
```
## embl:下载所有的脊椎动物，这步跳过
```bash
crabs db_download --source embl --database 'VRT*' --output embl_vrt.fasta --keep_original yes
```
## bold:下载鱼类数据
```bash
crabs db_download --source bold --database 'Mollusca|Arthropoda|Annelida' --output bold_benthno.fasta --keep_original yes
```
## mifish：整个数据库
```bash
crabs db_download --source mitofish --output mitofish.fasta --keep_original yes
```
## 命名系统：download ncbi taxanomy system
```bash
crabs db_download --source taxonomy
```
## 合并以上下载的所有序列
```bash
crabs db_merge --output all.fasta --uniq yes --input bold_benthno.fasta diqi.fasta
```
## 根据我们要用的前后引物对所有序列进行切割
```bash
crabs insilico_pcr --input elas_parts_12S.fasta --output Mifish_E_cut.fasta --fwd GTTGGTAAATCTCGTGCCAGC --rev CATAGTGGGGTATCTAATCCTAGTTTG  --error 5
```
## 由于很多序列已经切去了引物区或者引物区识别失败，有用的序列没有被保留，我们通过对齐的方法再次进行筛选和保留
```bash
crabs pga --input CRABS_ncbi_download.fasta --output Mifish_E_noPrimer.fasta --database Mifish_E_cut.fasta --fwd GTTGGTAAATCTCGTGCCAGC --rev CATAGTGGGGTATCTAATCCTAGTTTG --speed medium --percid 0.5 --coverage 0.5 --filter_method strict
```
## 核对并统一命名方式
```bash
crabs assign_tax --input elas_parts_16S.fasta --output 16S_assigned.tsv --acc2tax nucl_gb.accession2taxid --taxid nodes.dmp --name names.dmp --missing 16S_missing_taxa.tsv
```
## 去重复，三种方式，推荐选
```bash
crabs dereplicate --input all_assigned.tsv --output assigned_uniq.tsv --method uniq_species
```
## 筛选--去除过小、过大、寡碱基过多的序列
```bash
crabs seq_cleanup --input assigned_uniq.tsv --output assigned_uniq_filtered.tsv --minlen 100 --maxlen 1000 --maxns 0 --enviro yes --species yes --nans 0
```
## 输出：vsearch适用格式
```bash
crabs tax_format --input assigned_uniq_filtered.tsv --output vsearch_reference.fasta --format sintax
```
## 输出：dada2适用格式
```bash
crabs tax_format --input assigned_uniq_filtered.tsv --output dada2_reference.fasta --format dad
```
## 输出：makeblastdb适用格式
```bash
Rscript blast_reference.R -i assigned_uniq_filtered.tsv -o blast_reference.fasta
```
## 目标序列被下载比例
```bash
Rscript generate_stats.R -r assigned_uniq_filtered.tsv -q elas_parts.csv
```