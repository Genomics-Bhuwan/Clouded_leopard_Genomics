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

#### Note:
---
- The recommended number of SNPs per chromosomes for analysis with GONe or CurrentNe is between 50k to 100k.
- We therefore randomly subsampled 2 million filtered loci from across all the autosomes to achieve approximately 70k per autosomes.
- We then split the dataset into populations and ran CurrentNe using the default paramters.

```

#### Runnign CurrentNe using the subsampling stuff.

```bash
# Path to your actual original VCF
ORIGINAL_VCF="/shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/Clouded_leopard_27samples_autosomes_biallelic.vcf.gz"

# Target path for the subsampled file
MASTER_2M="/shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2/subsampled_2M.vcf"

echo "Extracting header..."
bcftools view -h "$ORIGINAL_VCF" > "$MASTER_2M"

echo "Randomly sampling 2,000,000 SNPs... this may take a moment."
bcftools view -H "$ORIGINAL_VCF" | shuf -n 2000000 >> "$MASTER_2M"

echo "Subsampling complete. Verification:"
ls -lh "$MASTER_2M"
``

#### Running currentNe finally
```bash
# Set variables again just to be sure
BASE_DIR="/shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/GONE2"
BINARY="/shared/jezkovt_bistbs_shared/Clouded_leopard_Genomics_Project/Clouded_leopard_GONe/currentNe/currentNe"
MASTER_VCF="${BASE_DIR}/subsampled_2M.vcf"

POPS=("China" "UK" "USA")
LISTS=("list_China.txt" "list_UK.txt" "list_USA.txt")

for i in "${!POPS[@]}"; do
    POP_NAME=${POPS[$i]}
    SAMPLE_LIST="${BASE_DIR}/${LISTS[$i]}"
    OUT_VCF="${BASE_DIR}/${POP_NAME}_final_input.vcf"
    
    echo "--- Processing $POP_NAME ---"
    
    # 1. Clean the sample list (removes hidden spaces/columns)
    awk '{print $1}' "$SAMPLE_LIST" > "${SAMPLE_LIST}.clean"
    
    # 2. Extract population samples from the 2M master file
    bcftools view -S "${SAMPLE_LIST}.clean" "$MASTER_VCF" --force-samples > "$OUT_VCF"
    
    echo "Running currentNe for $POP_NAME..."
    mkdir -p "${BASE_DIR}/Results_${POP_NAME}"
    
    # 3. Run the analysis
    $BINARY "$OUT_VCF" 18 \
        -t 16 \
        -v 1 \
        -o "${BASE_DIR}/Results_${POP_NAME}/${POP_NAME}_CNe_Results"
        
    echo "DONE with $POP_NAME"
done

```

