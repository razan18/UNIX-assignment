#UNIX Assignment

##Data Inspection

###Attributes of `fang_et_al_genotypes` and `snp_position`

To look at the first 10 lines and inspect the headers
```
head fang_et_al_genotypes.txt
```
To look at the number of columns
```
awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
``` 
To look at the number of words, lines, characters (bytes)
```
wc fang_et_al_genotypes.txt
```
To look at the file size
```
du -h fang_et_al_genotypes.txt
```
To look at file type
```
file fang_et_al_genotypes.txt
```

By inspecting this file I learned that:
Note: all previous code was repeated for the snp position file, just changing the file being selected in the command for inspection.

1. The fang et al had was very unorganized and I can't tell what the headers are from what the data is.
2. The fang et al file had 986 columns, snp position file had 15 columns.
3. The fang et al file had 2783 lines, 2744038 words, 11051939 characters. The snp position file had 984 lines, 13198 words, 82763 characters.
4. The fang et al file is 6.5M in size, the snp position file is 41K in size.
5. Both files are ASCII text.


##Data Processing

To extract column 3 which is 'position' and sort it based on the number of occurences of each term in that column
```
cut -f3 fang_et_al_genotypes.txt | sort | uniq -c
```
To sort snp position file based on the required columns
```
cut -f 1,3,4 snp_position.txt > 134col_snp_pos.txt
sort -k1,1 134col_snp_pos.txt > sorted_cut_snp_pos.txt 
```
To grab the header into a new file
```
head -n 1 fang_et_al_genotypes.txt > header.txt
```
###Maize Data

To match anything with 'ZMM' from the fang file and direct the standard out to a new file called maize_geno. Match 'ZMM' as all maize start with this.
```
grep 'ZMM' fang_et_al_genotypes.txt > maize_genotypes.txt
```

To combine the header with maize genotypes. The code after checks how the data looks like.
```
cat header.txt maize_genotypes.txt > header_maize_genotypes.txt

head -n 10 maize_genotypes.txt | cut -c -100 | column -t
```
To extract the groups and maize genotypes to a new file
```
cut -f 3-986 header_maize_genotypes.txt > snps_only_header_maize_genotypes.txt
```
To transpose data after extracting maize data
```
awk -f transpose.awk snps_only_header_maize_genotypes.txt > transposed_maize_genotypes.txt
```
Before joining, we have to sort the SNP_ID. This will sort based on column 1 and print the standard out into the new files. I also checked if they're sorted.
```
sort -k1,1 transposed_maize_genotypes.txt > sorted_maize_genotypes.txt
head -n 10 sorted_maize_genotypes.txt | cut -c -50 | column -t
```

To join sorted snp and maize 
```
join -1 1 -2 1 -t $'\t' sorted_cut_snp_pos.txt sorted_maize_genotypes.txt > join_maize_genotypes.txt
```
Sort increasing SNP position for maize
```
grep -v "multiple" join_maize_genotypes.txt | grep -v "unkown" | sort -k3,3n > increase_maize_genotypes.txt
```
Sort decreasing SNP position for maize
```
grep -v "multiple" join_maize_genotypes.txt | grep -v "unkown" | sort -k3,3nr > decrease_maize_genotypes.txt
```
Make a file for multiple snps in maize genotype
```
awk '$3 ~ /^multiple$/' join_maize_genotypes.txt > maize_multiple.txt
```
Make a file for unknown snps in maize genotype
```
awk '$3 ~ /^unknown$/' join_maize_genotypes.txt > maize_unknown.txt
```
Make directory to put all maize data files
```
mkdir maize_data
```
Loop to make individual chromosome files (chr1-10) based on increasing snp position with unknown ?
```
for i in {1..10}; do awk '$2== '$i'' increase_maize_genotypes.txt | sort -k3,3n > maize_data/maize_data_chr"$i"_increase.txt; done
```
Replace missing value in decreasing maize genotype with "-"
```
sed 's/?/-/g' decrease_maize_genotypes.txt > decrease_maize_genotype_dash.txt
```
Loop to make individual chromosome files (chr1-10) based on decreasing snp position with unknown "-"
```
for i in {1..10}; do awk '$2=='$i'' decrease_maize_genotype_dash.txt > maize_data/maize_chr"$i"_decrease.txt; done
```
###Teosinte Data
To match anything with 'ZMP' from the fang file and direct the standard out to a new file called teosinte_geno. Match 'ZMP' as all teosinte start with this.
```
grep 'ZMP' fang_et_al_genotypes.txt > teosinte_genotypes.txt
```
To combine the header with teosinte genotypes. The code after checks how the data looks like.
```
cat header.txt teosinte_genotypes.txt > header_teosinte_genotypes.txt

head -n 10 teosinte_genotypes.txt | cut -c -100 | column -t
```
To check the number of columns is 986
```
awk -F "\t" '{print NF; exit}' header_teosinte_genotypes.txt
```
To extract the groups and maize genotypes to a new file
```
cut -f 3-986 teosinte_genotypes.txt > snps_only_header_teosinte_genotypes.txt
```
To transpose data after extracting maize data
```
awk -f transpose.awk snps_only_header_teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
```
Before joining, we have to sort the SNP_ID. This will sort based on column 1 and print the standard out into the new files. I also checked if they're sorted.
```
sort -k1,1 transposed_teosinte_genotypes.txt > sorted_teosinte_genotypes.txt
head -n 10 sorted_teosinte_genotypes.txt | cut -c -50 | column -t
```
To join sorted snp and teosinte
```
join -1 1 -2 1 -t $'\t' sorted_cut_snp_pos.txt sorted_teosinte_genotypes.txt > join_teosinte_genotypes.txt
```
Sort increasing SNP position for teosinte
```
grep -v "multiple" join_teosinte_genotypes.txt | grep -v "unkown" | sort -k3,3n > increase_teosinte_genotypes.txt
```
Sort decreasing SNP position for teosinte
```
grep -v "multiple" join_teosinte_genotypes.txt | grep -v "unkown" | sort -k3,3nr > decrease_teosinte_genotypes.txt
```
Make a file for multiple snps in teosinte genotype
```
awk '$3 ~ /^multiple$/' join_teosinte_genotypes.txt > teosinte_multiple.txt
```
Make a file for unknown snps in teosinte genotype
```
awk '$3 ~ /^unknown$/' join_teosinte_genotypes.txt > teosinte_unknown.txt
```
Make directory to put all teosinte data files
```
mkdir teosinte_data
```
Loop to make individual chromosome files (chr1-10) based on increasing snp position with unknown ?
```
for i in {1..10}; do awk '$2== '$i'' increase_teosinte_genotypes.txt | sort -k3,3n > teosinte_data/teosinte_data_chr"$i"_increase.txt; done
```
Replace missing value in decreasing teosinte genotype with "-"
```
sed 's/?/-/g' decrease_teosinte_genotypes.txt > decrease_teosinte_genotypes_dash.txt
```
Loop to make individual chromosome files (chr1-10) based on decreasing snp position with unknown "-"
```
for i in {1..10}; do awk '$2=='$i'' decrease_teosinte_genotypes_dash.txt > teosinte_data/teosinte_chr"$i"_decrease.txt; done
```
