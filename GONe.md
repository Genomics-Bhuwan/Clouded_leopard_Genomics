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
# Define the original VCF as a variable for easy typing
ORIG_VCF="Clouded_leopard_27samples_autosomes_biallelic.vcf.gz"

# 1. China VCF
bcftools view -S list_China.txt $ORIG_VCF -Oz -o Clouded_China.vcf.gz

# 2. USA VCF
bcftools view -S list_USA.txt $ORIG_VCF -Oz -o Clouded_USA.vcf.gz

# 3. UK VCF
bcftools view -S list_UK.txt $ORIG_VCF -Oz -o Clouded_UK.vcf.gz
```

