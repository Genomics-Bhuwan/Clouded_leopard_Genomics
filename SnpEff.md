#### Use the pipeline below: https://github.com/pcingola/SnpEff/blob/master/examples/examples.sh
```bash
#SBATCH --job-name=CloudedLeopard_SnpEff
#SBATCH --cpus-per-task=8
#SBATCH --mem=80G
#SBATCH --time=48:00:00

set -euo pipefail

########################################
# VARIABLES
########################################

PROJECT=$HOME/Clouded_leopard_SNPeff
SNPEFF=${PROJECT}/snpEff

GENOME_ID="mNeoNeb1"
GENOME_NAME="Clouded_leopard"

GENOME_FASTA="${PROJECT}/GCF_028018385.1_mNeoNeb1.pri_genomic.fna"
GENOME_GFF="${PROJECT}/GCF_028018385.1_mNeoNeb1.pri_genomic.gff.gz"

INPUT_VCF="${PROJECT}/Clouded_leopard_27samples_RefSeq_Standardized.vcf.gz"

OUTPUT_VCF="${PROJECT}/Clouded_leopard_27samples_annotated.vcf.gz"
REPORT_HTML="${PROJECT}/snpeff_report.html"

########################################
# CREATE DATABASE DIRECTORY
########################################

mkdir -p ${SNPEFF}/data/${GENOME_ID}

########################################
# PREPARE GENOME FILES
########################################

echo "Preparing genome annotation files..."

gunzip -c ${GENOME_GFF} \
    > ${SNPEFF}/data/${GENOME_ID}/genes.gff

cp ${GENOME_FASTA} \
   ${SNPEFF}/data/${GENOME_ID}/sequences.fa

########################################
# UPDATE SNPEFF CONFIG
########################################

echo "Checking snpEff.config..."

if ! grep -q "^${GENOME_ID}\.genome" ${SNPEFF}/snpEff.config; then
    echo "${GENOME_ID}.genome : ${GENOME_NAME}" \
    >> ${SNPEFF}/snpEff.config
fi

########################################
# BUILD DATABASE
########################################

echo "Building SnpEff database..."

cd ${SNPEFF}

java -Xmx70g -jar snpEff.jar build \
    -gff3 \
    -v \
    ${GENOME_ID}

########################################
# ANNOTATE VARIANTS
########################################

echo "Annotating variants..."

java -Xmx32g -jar snpEff.jar ann \
    -v \
    -stats ${REPORT_HTML} \
    ${GENOME_ID} \
    ${INPUT_VCF} \
    | bgzip > ${OUTPUT_VCF}

########################################
# INDEX OUTPUT VCF
########################################

echo "Indexing annotated VCF..."

tabix -p vcf ${OUTPUT_VCF}

########################################
# CHECK OUTPUT
########################################

echo "Annotation complete."

echo ""
echo "Generated files:"
echo "  ${OUTPUT_VCF}"
echo "  ${OUTPUT_VCF}.tbi"
echo "  ${REPORT_HTML}"
echo ""

echo "Quick verification:"
bcftools view ${OUTPUT_VCF} | head -20
```

#### Classify the variants based on the impact or the consequence
- Synonymous
- Misense
- Loss of funcction.
  
```bash
# Define paths for cleaner execution
WORKING_DIR="/home/bistbs/Clouded_leopard_SNPeff"
SNPSIFT_JAR="${WORKING_DIR}/snpEff/SnpSift.jar"
JAVA_EXEC="${WORKING_DIR}/snpEff/jdk-21.0.2/bin/java"

INPUT_VCF="${WORKING_DIR}/Clouded_Leopard_Annotated.vcf"
OUTPUT_DIR="${WORKING_DIR}/Variants_by_Consequences"

# 1. Create the output directory if it doesn't exist
echo "Creating output directory..."
mkdir -p ${OUTPUT_DIR}

# 2. Filter for Missense Variants
echo "Extracting Missense variants..."
${JAVA_EXEC} -jar ${SNPSIFT_JAR} filter "ANN[*].EFFECT has 'missense_variant'" \
  ${INPUT_VCF} \
  > ${OUTPUT_DIR}/Clouded_leopard_missense_sites.vcf

# 3. Filter for Synonymous Variants
echo "Extracting Synonymous variants..."
${JAVA_EXEC} -jar ${SNPSIFT_JAR} filter "ANN[*].EFFECT has 'synonymous_variant'" \
  ${INPUT_VCF} \
  > ${OUTPUT_DIR}/Clouded_leopard_synonymous_sites.vcf

# 4. Filter for Loss of Function (LoF) & Splicing
echo "Extracting Loss of Function (LoF) & Splicing variants..."
${JAVA_EXEC} -jar ${SNPSIFT_JAR} filter "(ANN[*].EFFECT has 'transcript_ablation') | (ANN[*].EFFECT has 'splice_donor_variant') | (ANN[*].EFFECT has 'splice_acceptor_variant') | (ANN[*].EFFECT has 'stop_gained') | (ANN[*].EFFECT has 'frameshift_variant') | (ANN[*].EFFECT has 'inframe_insertion') | (ANN[*].EFFECT has 'inframe_deletion') | (ANN[*].EFFECT has 'splice_region_variant')" \
  ${INPUT_VCF} \
  > ${OUTPUT_DIR}/Clouded_leopard_lof_sites.vcf

# 5. Filter for Intergenic Variants
echo "Extracting Intergenic variants..."
${JAVA_EXEC} -jar ${SNPSIFT_JAR} filter "ANN[*].EFFECT has 'intergenic_region'" \
  ${INPUT_VCF} \
  > ${OUTPUT_DIR}/Clouded_leopard_intergenic_sites.vcf

# -----------------------------------------------------------------
# Verification Block
# -----------------------------------------------------------------
echo "---------------------------------------------------"
echo "Filtering Complete! Total variant counts extracted:"
echo "---------------------------------------------------"
echo -n "Missense Sites:   " && grep -v "^#" ${OUTPUT_DIR}/Clouded_leopard_missense_sites.vcf | wc -l
echo -n "Synonymous Sites: " && grep -v "^#" ${OUTPUT_DIR}/Clouded_leopard_synonymous_sites.vcf | wc -l
echo -n "LoF/Splicing:     " && grep -v "^#" ${OUTPUT_DIR}/Clouded_leopard_lof_sites.vcf | wc -l
echo -n "Intergenic Sites: " && grep -v "^#" ${OUTPUT_DIR}/Clouded_leopard_intergenic_sites.vcf | wc -l
echo "---------------------------------------------------"
```
