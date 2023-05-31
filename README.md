<p align="center">
   <img src=https://github.com/nataliering/Dogstails/assets/23463469/243e064b-17c3-4aae-bafc-07cabd66d758 height="300"/>
</p>




# Rapid metagenomic sequencing for diagnosis and antimicrobial sensitivity prediction of canine bacterial infections 
Code and commands used during the development of our rapid diagnostics pipeline for canine urine and skin swab samples

## Abstract
Antimicrobial resistance is one of the greatest current threats to human and animal health. There is an urgent need to ensure that antimicrobials are used appropriately to limit the emergence and impact of resistance. In the human and veterinary healthcare setting, traditional culture and antimicrobial sensitivity testing is typically conducted, requiring 48-72 h to identify appropriate antibiotics for treatment. In the meantime, broad-spectrum antimicrobials are often used, which may be ineffective or impact non-target commensal bacteria. Here, we present a rapid diagnostics pipeline, involving metagenomic Nanopore sequencing directly from clinical urine and skin samples of dogs. We have optimised this pipeline to be versatile and easily implementable in a clinical setting, with the potential for future adaptation to different sample types and animals. Using our approach, we can identify the bacterial pathogen present within 5 hours, in some cases detecting species which are difficult to culture. For urine samples, we can predict antibiotic sensitivity with up to 95% accuracy. Skin swabs exhibited lower bacterial abundance and higher host DNA, confounding antibiotic sensitivity prediction; an additional host depletion step will likely be required during the processing of these, and other types of samples with high levels of host cell contamination. In summary, our pipeline represents an important step towards the design of individually tailored veterinary treatment plans on the same day as presentation, facilitating the effective use of antibiotics and promoting better antimicrobial stewardship.  


## Commands for tools mentioned in manuscript
Each of the tools we used can be further optimised; we tended to use the default settings in most cases, often exactly as recommended in the tool's README.

### Downloading relevant nanopore datasets in fastq format
**[fasterq-dump (download of reads from SRA)](https://github.com/ncbi/sra-tools)**  
`fasterq-dump ACCESSION_NUMBER --gzip`

### Adaptor trimming
**[Porechop](https://github.com/rrwick/Porechop)**  
`porechop -i INPUT.fastq -o OUTPUT.fastq`

### Random subsampling
**[Rasusa](https://github.com/mbhall88/rasusa)**                                                                                                                                                 
`rasusa --bases NUMBER_OF_BASES --input INPUT.fastq.gz > OUTPUT.NUMBER_OF_BASES.fastq.gz`

### Simulating nanopore reads of circa 2020 quality
**[Badread](https://github.com/rrwick/Badread)**                                                                                                                                                             
`badread simulate --reference SPECIES_REFERENCE_GENOME.fna --quantity 25x --error_model nanopore2020 --qscore_model nanopore2020 --length 5000,5000 --identity 90,98,5   | gzip > SPECIES_badreads.fastq.gz`

### Taxonomic classification
**[Kraken2](https://github.com/DerrickWood/kraken2)**                                                                                                                                             
`kraken2 --db DATABASE READS.fastq.gz --threads 8 --confidence 0/0.05/0.1 --report READS.DATABASE.CONFIDENCE.report.tsv --output READS.DATABASE.CONFIDENCE.txt`

**[EPI2ME](https://epi2me.nanoporetech.com/)**                                                                                                                                                                       
EPI2ME can be used via a GUI or browser, or you can submit reads to the server via the interactive command line agent (CLI):                                                                     
`epi2me workflows run INPUT_FOLDER`

### Genome assembly and annotation
**[Flye](https://github.com/fenderglass/Flye)**  
`flye --meta --threads 8 --out-dir OUTPUT_DIRECTORY --nano-raw INPUT.fastq`

**[Prokka](https://github.com/tseemann/Prokka)**                                                                                                            
`prokka --compliant --genus GENUS --species SPECIES --metagenome --cpus 8 --outdir PROKKA_OUTPUT --prefix PROKKA_OUTPUT INPUT_ASSEMBLY.fasta`

### AMR prediction tools                                                                                                                                    
**[ABRicate](https://github.com/tseemann/ABRicate)**                                                                                                                                                               
`abricate FLYE_OUTPUT.fasta > abricate.tsv`

**[AMRFinderPlus](https://github.com/ncbi/amr)**                                                                                       
`amrfinder -n PROKKA_OUTPUT.fna --organism ORGANISM -p PROKKA_OUTPUT.faa -g PROKKA_OUTPUT.gff --output $patient.ecoli.amrfinder.plus --plus --annotation_format prokka --threads 8`

**Separating reads from different channels after sequencing**                                                                                                                      
`for i in seq 1 1 256; do grep -A 3 --no-group-separator "ch=$1 " reads.fastq.gz > reads.1-256.fastq.gz; done`

