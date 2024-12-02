# reference-database-genertors/CRABS
> original web:https://github.com/gjeunen/reference_database_creator
#according to your species list, download all sequences from nucleotide database in ncbi which includes amplicons, wgs and so on.
...
crabs db_download --source ncbi --database nucleotide --query '12S[All Fields] AND ("1"[SLEN] : "50000"[SLEN])' --species elas_parts.csv --output elas_parts.fasta --keep_original yes --email fangliufree@gmail.com --batchsize 50000
...

#download all vertebrate sequences from embl
crabs db_download --source embl --database 'VRT*' --output embl_vrt.fasta --keep_original yes 

#download all fish sequences from bold
crabs db_download --source bold --database 'Mollusca|Arthropoda|Annelida' --output bold_benthno.fasta --keep_original yes

#download whole mifish
crabs db_download --source mitofish --output mitofish.fasta --keep_original yes

#download ncbi taxanomy system
crabs db_download --source taxonomy

#merge all fasta files from ncbi, embl, bold and mifish
crabs db_merge --output all.fasta --uniq yes --input bold_benthno.fasta diqi.fasta

#select sequences with primers we employed, trim off the primers, and keep only targeted region 
crabs insilico_pcr --input elas_parts_12S.fasta --output Mifish_E_cut.fasta --fwd GTTGGTAAATCTCGTGCCAGC --rev CATAGTGGGGTATCTAATCCTAGTTTG  --error 5

#according to what we get from last step, align all other sequences without primers and keep the region we are interested, this could take a while
crabs pga --input CRABS_ncbi_download.fasta --output Mifish_E_noPrimer.fasta --database Mifish_E_cut.fasta --fwd GTTGGTAAATCTCGTGCCAGC --rev CATAGTGGGGTATCTAATCCTAGTTTG --speed medium --percid 0.5 --coverage 0.5 --filter_method strict

#assign the correct taxanomy 
crabs assign_tax --input elas_parts_16S.fasta --output 16S_assigned.tsv --acc2tax nucl_gb.accession2taxid --taxid nodes.dmp --name names.dmp --missing 16S_missing_taxa.tsv

crabs assign_tax --input elas_parts_12S.fasta --output 12S_assigned.tsv --acc2tax nucl_gb.accession2taxid --taxid nodes.dmp --name names.dmp --missing 12S_missing_taxa.tsv

crabs assign_tax --input elas_parts_COI.fasta --output COI_assigned.tsv --acc2tax nucl_gb.accession2taxid --taxid nodes.dmp --name names.dmp --missing COI_missing_taxa.tsv

crabs assign_tax --input elas_parts_NADH2.fasta --output NADH2_assigned.tsv --acc2tax nucl_gb.accession2taxid --taxid nodes.dmp --name names.dmp --missing NADH2_missing_taxa.tsv


#keep unique species and all unique sequences belong to these species
crabs dereplicate --input all_assigned.tsv --output assigned_uniq.tsv --method uniq_species

#remove smaller and larger sequences and sequences without metadata
crabs seq_cleanup --input assigned_uniq.tsv --output output.tsv --minlen 100 --maxlen 1000 --maxns 0 --enviro yes --species yes --nans 0

#this is optional, include or exclude some sequences you want to exclude or include
crabs db_subset --input input.tsv --output output.tsv --database userlist.txt --subset include

#generate a vsearch reference database
crabs tax_format --input output.tsv --output vsearch_reference.fasta --format sintax
#generate a dada2 reference database
crabs tax_format --input assigned_uniq_cleaned.tsv --output dada2_reference.fasta --format dad

#generate a blast database step1
Rscript blast_reference.R -i output.tsv -o blast_reference.fasta

#generate a blast database step2
makeblastdb -in blast_reference.fasta -parse_seqids -dbtype nucl -blastdb_version 5

#generate a stats for this database
Rscript generate_stats.R -r output.tsv -q diqi.csv
