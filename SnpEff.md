#### Use the pipeline below: https://github.com/pcingola/SnpEff/blob/master/examples/examples.sh
#!/bin/bash
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
