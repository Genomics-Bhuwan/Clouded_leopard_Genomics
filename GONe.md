#### I want to assess the effective population size across three different countries(China, USA and Thailand).
```bash
# Create United States list
printf "NN114296\nNN114297\nNN114393\nNN115950\nNN190240\nSRR13774417" > list_USA.txt

# Create China list
printf "SAMN32324407\nSAMN32324408\nSAMN32324409\nSAMN32324410\nSAMN32324411\nSAMN32324426\nSAMN32324430" > list_China.txt

# Create United Kingdom list
printf "SAMN32324412\nSAMN32324413\nSAMN32324414\nSAMN32324415\nSAMN32324416\nSAMN32324417\nSAMN32324418\nSAMN32324419\nSAMN32324420\nSAMN32324421\nSAMN32324422\nSAMN32324423\nSAMN32324424\nSAMN32324425" > list_UK.txt
```
#### Step 1. Split the biallelic vcf of the clouded leopard with all the 27 samples.

```bash
mkdir -p /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/China
mkdir -p /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/UK
mkdir -p /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/USA

#### Subsetting each population.
bcftools view -S /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/list_China.txt \
  -Oz -o /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/China/China.vcf.gz \
  /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/Clouded_leopard_27samples_autosomes_biallelic.vcf.gz

bcftools view -S /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/list_UK.txt \
  -Oz -o /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/UK/UK.vcf.gz \
  /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/Clouded_leopard_27samples_autosomes_biallelic.vcf.gz

bcftools view -S /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/list_USA.txt \
  -Oz -o /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/USA/USA.vcf.gz \
  /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/Clouded_leopard_27samples_autosomes_biallelic.vcf.gz


####index each file
bcftools index /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/China/China.vcf.gz
bcftools index /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/UK/UK.vcf.gz
bcftools index /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/USA/USA.vcf.gz
```

#### Note:
---
- The recommended number of SNPs per chromosomes for analysis with GONe or CurrentNe is between 50k to 100k.
- We therefore randomly subsampled 2 million filtered loci from across all the autosomes to achieve approximately 70k per autosomes.
- We then split the dataset into populations and ran CurrentNe using the default paramters.

```

#### Runnign CurrentNe using the subsampling stuff.

```bash

Run currentNe
For each population, run:


/shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/China/plink/currentNe -k 0 -s 70000 -o China_currentNe \
  /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/China/China.vcf.gz 18

/shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/China/plink/currentNe -k 0 -s 70000 -o UK_currentNe \
  /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/UK/UK.vcf.gz 18

/shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/China/plink/currentNe -k 0 -s 70000 -o USA_currentNe \
  /shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/USA/USA.vcf.gz 18
```
