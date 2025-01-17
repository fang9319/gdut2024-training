## crabs新版
### （1）ncbi:根据准备好的物种名录，在ncbi上下载序列，一般使用nucleotide数据库.
```bash 
crabs --download-ncbi --query '(12S[All Fields] AND ("100"[SLEN] : "25000"[SLEN]))' --species split_part_aa.txt --output ncbi_chondrichthyes.fasta --email johndoe@gmail.com --database nucleotide
```
### （2）embl:下载所有的脊椎动物，这步跳过
```bash
crabs --download-embl --taxon 'ver*' --output crabs_testing/embl_ver.fasta
```
### （3）bold:下载鱼类数据
```bash
crabs --download-bold --taxon 'Elasmobranchii' --output bold_Chondrichthyes.fasta
```
### （4）mifish：整个数据库
```bash
crabs --download-mitofish --output mitofish.fasta
```
### （5）命名系统：download ncbi taxanomy system 跳过这步
```bash
crabs --download-taxonomy 
```
### （6）导入
```bash
crabs --import --import-format bold --input bold_Chondrichthyes.fasta --names ../names.dmp --nodes ../nodes.dmp --acc2tax ../nucl_gb.accession2taxid --output crabs_bold.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
crabs --import --import-format ncbi --input ncbi_Chondrichthyes.fasta --names ../names.dmp --nodes ../nodes.dmp --acc2tax ../nucl_gb.accession2taxid --output crabs_ncbi.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
crabs --import --import-format mitofish --input mitofish.fasta --names ../names.dmp --nodes ../nodes.dmp --acc2tax ../nucl_gb.accession2taxid --output mitofish.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
crabs --import --import-format ncbi --input ../mine.fasta --names ../names.dmp --nodes ../nodes.dmp --acc2tax ../nucl_gb.accession2taxid --output mine.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
```
### (7)合并以上下载的所有序列，为了节省时间不加入mitofish
```bash
crabs --merge --input '../crabs_bold.txt;crabs_ncbi.txt;../mine.txt;mitofish.txt' --uniq --output merged.txt
```
### (8)根据我们要用的前后引物对所有序列进行切割
```bash
crabs --in-silico-pcr --input merged.txt --output insilico.txt --forward TCAACCAACCACAAAGACATTGGCAC --reverse AAGATTACAAAAGCGTGGGC --threads 2
```
### (9)由于很多序列已经切去了引物区或者引物区识别失败，有用的序列没有被保留，我们通过对齐的方法再次进行筛选和保留
```bash
crabs --pairwise-global-alignment --input merged.txt --amplicons insilico.txt --output aligned.txt --forward TCAACCAACCACAAAGACATTGGCAC --reverse AAGATTACAAAAGCGTGGGC --size-select 10000 --percent-identity 0.85 --coverage 0.85
```
### (10)去重复，三种方式，推荐选
```bash
crabs --dereplicate --input aligned.txt --output dereplicated.txt --dereplication-method 'unique_species'
```
### (11)筛选--去除过小、过大、寡碱基过多的序列
```bash
crabs --filter --input dereplicated.txt --output filtered.txt --minimum-length 100 --maximum-length 400 --maximum-n 1 --environmental --no-species-id --rank-na 2
```
### (12)筛选自己感兴趣的物种
```bash
crabs --subset --input filtered.txt --output subset.txt --include 'Chondrichthyes'
```
### (13)输出
```bash
for format in sintax.fasta blast-notax.fasta; do crabs --export --input subset.txt --output chondrichthyes_${format} --export-format ${format%%.*}; done
```
### (14)数据库范围
```bash
crabs --amplicon-length-figure --input subset.txt --output figures/amplicon-length-figure.png --tax-level 4
```
### (15)目标序列被下载比例
```bash
crabs --completeness-table --input subset.txt --output figures/completeness.txt --names ../names.dmp --nodes ../nodes.dmp --species ../species-list.csv
```
### (16) 预测扩增效果
```bash
crabs --amplification-efficiency-figure --input merged.txt --amplicons subset.txt --forward TCAACCAACCACAAAGACATTGGCAC --reverse AAGATTACAAAAGCGTGGGC --output figures/amplification-efficiency.png --tax-group Carcharhiniformes
```
### (17) 拷贝到自己的文件夹
```bash
scp -P 8080 -r dell@10.24.22.176:/media/dell/eDNA2/db-training/lzf-COI/figures \
 ~/Downloads
```