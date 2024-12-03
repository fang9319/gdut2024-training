## crabs新版
### （1）ncbi:根据准备好的物种名录，在ncbi上下载序列，一般使用nucleotide数据库.
```bash 
crabs --download-ncbi --query '("Chondrichthyes"[Organism] OR Chondrichthyes[All Fields]) AND (mitochondrion[filter] AND ("100"[SLEN] : "25000"[SLEN]))' --species input.txt --output crabs_testing/ncbi_chondrichthyes.fasta --email johndoe@gmail.com --database nucleotide
```
### （2）embl:下载所有的脊椎动物，这步跳过
```bash
crabs --download-embl --taxon 'ver*' --output crabs_testing/embl_ver.fasta
```
### （3）bold:下载鱼类数据
```bash
crabs --download-bold --taxon Chondrichthyes --output crabs_testing/bold_Chondrichthyes.fasta; done
```
### （4）mifish：整个数据库
```bash
crabs --download-mitofish --output crabs_testing/mitofish.fasta
```
### （5）命名系统：download ncbi taxanomy system
```bash
crabs db_download --source taxonomy
```
### （6）导入
```bash
crabs --import --import-format bold --input crabs_testing/bold_Elasmobranchii.fasta --names crabs_testing/names.dmp --nodes crabs_testing/nodes.dmp --acc2tax crabs_testing/nucl_gb.accession2taxid --output crabs_testing/crabs_bold.txt --ranks 'superkingdom;phylum;class;order;family;genus;species'
```
### (7)合并以上下载的所有序列
```bash
crabs --merge --input 'crabs_testing/crabs_bold.txt;crabs_testing/crabs_mitofish.txt;crabs_testing/crabs_ncbi.txt' --uniq --output crabs_testing/merged.txt
```
### (8)根据我们要用的前后引物对所有序列进行切割
```bash
crabs --in-silico-pcr --input crabs_testing/merged.txt --output crabs_testing/insilico.txt --forward GACCCTATGGAGCTTTAGAC --reverse CGCTGTTATCCCTADRGTAACT
```
### (9)由于很多序列已经切去了引物区或者引物区识别失败，有用的序列没有被保留，我们通过对齐的方法再次进行筛选和保留
```bash
crabs --pairwise-global-alignment --input crabs_testing/merged.txt --amplicons crabs_testing/insilico.txt --output crabs_testing/aligned.txt --forward GACCCTATGGAGCTTTAGAC --reverse CGCTGTTATCCCTADRGTAACT --size-select 10000 --percent-identity 0.95 --coverage 0.95
```
### (10)去重复，三种方式，推荐选
```bash
crabs --dereplicate --input crabs_testing/aligned.txt --output crabs_testing/dereplicated.txt --dereplication-method 'unique_species'
```
### (11)筛选--去除过小、过大、寡碱基过多的序列
```bash
crabs --filter --input crabs_testing/dereplicated.txt --output crabs_testing/filtered.txt --minimum-length 100 --maximum-length 300 --maximum-n 1 --environmental --no-species-id --rank-na 2
```
### (12)输出
```bash
for format in sintax.fasta rdp.fasta idt-fasta.fasta idt-text.txt; do crabs --export --input crabs_testing/subset.txt --output crabs_testing/chondrichthyes_${format} --export-format ${format%%.*}; done
```
### (13)数据库范围
```bash
crabs --amplicon-length-figure --input crabs_testing/subset.txt --output crabs_testing/amplicon-length-figure.png --tax-level 4
```
### （14）建个进化树
```bash
crabs --phylogenetic-tree --input crabs_testing/subset.txt --output crabs_testing/phylo --tax-level 4 --species 'Carcharodon carcharias+Squalus acanthias'
```
### （15）目标序列被下载比例
```bash
crabs --completeness-table --input crabs_testing/subset.txt --output crabs_testing/completeness.txt --names crabs_testing/names.dmp --nodes crabs_testing/nodes.dmp --species 'Carcharodon carcharias+Squalus acanthias'
```

