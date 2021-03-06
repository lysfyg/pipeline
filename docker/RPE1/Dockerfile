FROM smei/mosaicatcher-pipeline

MAINTAINER Sascha Meiers meiers@embl.de
LABEL version="0.2"
LABEL containes.data="RPE-1"

# Install wget
RUN mkdir -p bam/RPE1-WT \
    && cd bam/RPE1-WT \
    && apt-get update \
    && apt-get install --no-install-recommends -y wget \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/*


# Download & prepare SNV call set
RUN mkdir snv_sites_to_genotype \
    && mkdir tmp \
    && cd tmp \
    && for x in $(seq 1 22) X; \
        do \
            echo "${x} chr${x}" >> rename-chrs; \
            echo "downloading chromosome $x"; \
            wget -q ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/supporting/GRCh38_positions/ALL.chr${x}_GRCh38_sites.20170504.vcf.gz; \
            echo "ALL.chr${x}_GRCh38_sites.20170504.vcf.gz" >> file-list; \
        done \
    && bcftools concat -Oz -o tmp.vcf.gz -f file-list \
    && bcftools annotate --rename-chrs rename-chrs tmp.vcf.gz \
        | bgzip \
        > ../snv_sites_to_genotype/ALL.chr1-22plusX_GRCh38_sites.20170504.renamedCHR.vcf.gz \
    && tabix ../snv_sites_to_genotype/ALL.chr1-22plusX_GRCh38_sites.20170504.renamedCHR.vcf.gz \
    && cd .. \
    && rm -rf tmp


# Download RPE-1 data from ENA
RUN mkdir -p bam/RPE1-WT/all \
    && cd bam/RPE1-WT/all \
    && for x in $(grep RPE1WT ../../../docker/ena_files.txt); \
        do \
        echo "downloading ${x}"; \
        wget -q ${x}; \
        wget -q ${x%.bai}; \
        done

# Setup the set of selected RPE-1 cells
RUN mkdir -p bam/RPE1-WT/selected \
    && cd bam/RPE1-WT/selected \
    && for x in 01 02 03 04 05 06 07 09 10 11 12 13 14 15 16 17 18 19 21 22 23 \
                24 25 26 28 29 30 31 32 34 35 36 37 38 39 40 41 42 45 47 48 49 \
                50 51 52 53 54 55 56 57 59 62 63 64 65 66 67 68 69 70 71 72 73 \
                74 76 77 78 80 81 83 84 85 86 89 90 91 92 93 94 95; \
        do \
        ln /pipeline/bam/RPE1-WT/all/RPE1WTPE204${x}*.bam; \
        ln /pipeline/bam/RPE1-WT/all/RPE1WTPE204${x}*.bam.bai; \
        done

# Download the reference genome
RUN cd /\
    && wget -q ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/GCA_000001405.15_GRCh38/seqs_for_alignment_pipelines.ucsc_ids/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.gz \
    && gunzip GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.gz \
    && mv GCA_000001405.15_GRCh38_no_alt_analysis_set.fna reference.fa \
    && samtools faidx reference.fa

# Add external data for package BSgenome.Hsapiens.UCSC.hg38
RUN cd /tmp \
    && wget -q https://bioconductor.org/packages/3.6/data/annotation/src/contrib/BSgenome.Hsapiens.UCSC.hg38_1.4.1.tar.gz \
    && tar -xf BSgenome.Hsapiens.UCSC.hg38_1.4.1.tar.gz  \
    && mv BSgenome.Hsapiens.UCSC.hg38/inst/extdata/single_sequences.2bit /usr/local/lib/R/site-library/BSgenome.Hsapiens.UCSC.hg38/extdata/single_sequences.2bit \
    && rm -rf BSgenome.Hsapiens.UCSC.hg38*


# Specify the SNV call set within Snake.config-singularity.json
RUN cd /pipeline \
    && awk '/snv_sites_to_genotype/ { \
            print "    \"snv_sites_to_genotype\" : \"snv_sites_to_genotype/ALL.chr1-22plusX_GRCh38_sites.20170504.renamedCHR.vcf.gz\","; \
            next} \
        1==1 {print}' Snake.config-singularity.json \
        > Snake.config-singularity-rpe1.json
