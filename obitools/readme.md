## obitools 4 建立数据库，使用obipcr
### ncbi下载数据
```bash
while read species; do
    esearch -db nucleotide -query "$species[Organism] AND COI[All Fields]" | efetch -format fasta > "${species// /_}_COI.fasta" || echo "No COI sequences found for $species"
done < species_list.txt
```
### 合并
```bash
cat *.fasta > combined_sequences.fasta
```
### 导入
```bash
obiimport --input-format fasta combined_sequences.fasta > formatted_sequences.obi
```
### 序列命名
```bash
obiannotate --taxonomy "formatted_sequences.obi" \
    --taxonomy-db "path_to_ncbi_taxonomy_database" \
    --output "annotated_sequences.obi"
```
### Index the Database
```bash
obiindex --input "annotated_sequences.obi" \
         --output "reference_database.index"
```

