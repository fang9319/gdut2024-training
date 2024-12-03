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

```bash
obiimport --input-format fasta combined_sequences.fasta > reference_data.obi
```

