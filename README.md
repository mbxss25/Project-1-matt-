#Project1 - group 6 - Sequence assemblies and quality control - Matt Loose. 

#Introduction: Group generated the strains of micro-organisms, however various genetic changes to the genomes were done but location of the changes were unknown. The long read data was mostly present. In order to know the changes assemblies were required. This respository consist of the codes for generating the assemblies from the long and short reads, along with the quality analysis on them.

#Code are for generating:
-Short read assembly
-long read assembly
-A hybrid short and long read assembly

#Following codes for Quality control of assemblies-
BUSCO
QUAST

#Following codes for visualization and annotation of assemblies.
-Flye
-Prokka
-Genovi
#————————————————————————————————————————--------------------------------------------------------------------------------#
#This repository is for the samples allocated to group 6. i.e Group 6-sample 6 - illuminate S6 - Nanopore Barcode 06.

#The data is provided in HPC at-

/workhere/students_2023/Matt_resources

-subfolders 
short reads (illumina data)
Long reads (Nanoporedata)

#most important things before running the SLURM script make sure the script is executable using "chmod +x script_name.sh"

#——————————————————————————————————————————————————-------------------------------------------------------------------------#
#installation and activation of nanoplot in conda enviroment
conda create –n nanoplot_test –c bioconda nanoplot

conda install -c bioconda nanoplot
#recommanded installtion using sbatch


#!/bin/bash


#SBATCH --job-name=grp6_nanoplot
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/mbxss25/slurm-%x-%j.out                 #replace "mbxss25 "with your username

source $HOME/.bash_profile

#activate conda environment
conda activate /shared/conda/shared                                   #path of the conda environment located

# your command
NanoPlot


conda deactivate

#The expected result is creating the conda environment, installation and activation of the nanoplot within the conda environment.
#-------------------------------------------------------------------------------------------------------------------------------#

#Using nanoplot for checking the samples provided. The following code is for the illumina short reads provided in the dataset. The samples are differentiated between R1 and R2 as Forward and reverse. 

#nanoplot short reads (R1 and R2) 

#!/bin/bash

#SBATCH --job-name=short_reads_nanoplot
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/payya4/slurm-%x-%j.out                                   # Replace 'payya4' with your username
#SBATCH --error=/shared/home/payya4/slurm-%x-%j.err                                    # Replace 'payya4' with your username

source $HOME/.bash_profile

#activate conda env
conda activate /shared/conda/shared                                                    #path of the conda environment located

#run nanoplot on short reads (R1)
NanoPlot --fastq /workhere/students_2023/group6/illumina_short/R1_files/*.fastq.gz \   #path of the files located
         --threads 8 \
         --plots kde dot \
         -o /workhere/students_2023/group6/nano_qc_R1_short                            #path of the location where results wanted.

#run nanoplot on short reads (R2)
NanoPlot --fastq /workhere/students_2023/group6/illumina_short/R2_files/*.fastq.gz \   #path of the files located
         --threads 8 \
         --plots kde dot \
         -o /workhere/students_2023/group6/nano_qc_R2_short                             #path of the location where results wanted.

#deactivate conda env
conda deactivate

#The expected result running this code is to generates quality control plots for these reads, which provides insights into read length distribution, quality scores, and other relevant metrics which are useful for assessing the quality of sequencing data.
#To see the results code following on terminal

python3 -m http.server 12121

#on web input ip of HPC, with server. e.g http://10.102.161.12:12121

#----------------------------------------------------------------------------------------#

#pass long reads

#!/bin/bash


#SBATCH --job-name=fastq_pass_long
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out                              #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err                               #replace "mbxjk6" with your username

source $HOME/.bash_profile

#activate conda env
conda activate /shared/conda/shared

#run nanoplot on long reads (pass)
NanoPlot --fastq /workhere/students_2023/group6/nanopore_long/* \
         --threads 8 \
         --plots kde dot \
         -o /workhere/students_2023/group6/nano_qc_long_pass

#deactivte conda env
conda deactivate


#----------------------------------------------------------------------------------------#

#fails long reads

#!/bin/bash

#SBATCH --job-name=fastq_fails_long
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/payya4/slurm-%x-%j.out     #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-%x-%j.err      #replace "payya4" with your username

source $HOME/.bash_profile

#activate conda env
conda activate /shared/conda/shared

#run nanoplot on long reads
NanoPlot --fastq /workhere/students_2023/group6/fastq_fails_long/* \
         --threads 8 \
         --plots kde dot \
         -o /workhere/students_2023/group6/nano_qc_long_fails

#deactivate conda env
conda deactivate

#----------------------------------------------------------------------------------------#
##Merging reads
Merging reads is the best practice in reproducibility, it is good for quality control and process steps, such as trimming seq and filtering low-quality reads, work on a single file may simplify the setup of pipelines, espscially when using tools that are not designed to handle paired files separately or concurrently.

#!/bin/bash

#SBATCH --job-name=merged_reads
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out   #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err    #replace "mbxjk6" with your username



# Merge long reads (pass)
cat /workhere/students_2023/group6/nanopore_long/* > \                                #path of the files located
/workhere/students_2023/group6/merged_long_reads_pass.fastq.gz                        #location where output needed.


# Merge long reads (fail)
cat /workhere/students_2023/group6/fastq_fails_long/* > \
/workhere/students_2023/group6/merged_long_reads_fail.fastq.gz


# Merge long reads (pass and fail)
cat /workhere/students_2023/group6/nanopore_long/* /workhere/students_2023/group6/fastq_fails_long/* > \
/workhere/students_2023/group6/merged_long_reads_pf.fastq.gz


# Merge short reads R1
cat /workhere/students_2023/group6/illumina_short/R1_files/* > \
/workhere/students_2023/group6/merged_R1_short.fastq.gz


# Merge short reads R2
cat /workhere/students_2023/group6/illumina_short/R2_files/* > \
/workhere/students_2023/group6/merged_R2_short.fastq.gz

#the expected output is to merge the multiple sequencing read files provided into a single files for easier management and subsequent analysis
#------------------------------------------------------------------------------------------------------------------------------------------------#
##Minimap (for long assembly)
This script is written to perform long read assembly using Minimap2, followed by assembly construction with Miniasm, and conversion of fil format into fasta file.

#long reads pass

#!/bin/bash

#SBATCH --job-name=minimap_long_assembly
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out             #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err              #replace "mbxjk6" with your username


source $HOME/.bash_profile


# Activate conda environment
conda activate /shared/conda/shared

# Assembly with minimap2
minimap2 -t 8 -x ava-ont /workhere/students_2023/group6/merged_long_reads_pass.fastq.gz \                  #path of your files located
| gzip -1 > /workhere/students_2023/group6/minimap_long_pass/minimap_long_pass.paf                         #path for output

# Miniasm
#miniasm -f \
#/workhere/students_2023/group6/merged_long_reads_pass.fastq.gz \
#/workhere/students_2023/group6/minimap_long_pass/"long_assembly.paf.gz" INSERT > /workhere/students_2023/group6/minimap_long_pass/"long_assembly_pass.gfa"

# Convert the assembly to FASTA format
#awk '/^S/{print ">"$2"\n"$3}' /workhere/students_2023/group6/long_assembly.gfa > /workhere/students_2023/group6/long_assembly.fasta

# Deactivate conda environment
conda deactivate
#------------------------------------------------------------------------------------------------------------------------------------------------#


#!/bin/bash

#SBATCH --job-name=minimap_long_pass
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-%x-%j.out                  #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-%x-%j.err                   #replace "payya4" with your username



source $HOME/.bash_profile


# Activate conda environment
conda activate /shared/conda/shared

# Assembly with minimap2
minimap2 -t 8 -x ava-ont /workhere/students_2023/group6/merged_long_reads_pass.fastq.gz /workhere/students_2023/group6/merged_long_reads_pass.fastq.gz | gzip -1 > /workhere/students_2023/group6/minimap_r$

# Miniasm
miniasm -f /workhere/students_2023/group6/merged_long_reads_pass.fastq.gz  /workhere/students_2023/group6/minimap_results/minimap_long_pass/minimap_long_pass.paf.gz > /workhere/students_2023/group6/minim$

# Convert the assembly to FASTA format
awk '/^S/{print ">"$2"\n"$3}' /workhere/students_2023/group6/minimap_results/minimap_long_pass/minimap_long_pass.gfa > /workhere/students_2023/group6/minimap_results/minimap_long_pass/minimap_long_pass.f$

# Deactivate conda environment
conda deactivate
#------------------------------------------------------------------------------------------------------------------------------------------------#

#minimap pass and fail

#!/bin/bash

#SBATCH --job-name=minimap_long_passfail
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-%x-%j.out                  #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-%x-%j.err                  #replace "payya4" with your username




source $HOME/.bash_profile


# Activate conda environment
conda activate /shared/conda/shared

# Assembly with minimap2
minimap2 -t 8 -x ava-ont /workhere/students_2023/group6/merged_long_reads_pf.fastq.gz /workhere/students_2023/group6/merged_long_reads_pf.fastq.gz | gzip -1 > /workhere/students_2023/group6/minimap_resul$

# Miniasm
miniasm -f /workhere/students_2023/group6/merged_long_reads_pf.fastq.gz  /workhere/students_2023/group6/minimap_results/minimap_long_pf/minimap_long_pf.paf.gz > /workhere/students_2023/group6/minimap_res$

# Convert the assembly to FASTA format
awk '/^S/{print ">"$2"\n"$3}' /workhere/students_2023/group6/minimap_results/minimap_long_pf/minimap_long_pf.gfa > /workhere/students_2023/group6/minimap_results/minimap_long_pf/minimap_long_pf.fasta

# Deactivate conda environment
conda deactivate

#The expected put of these codes are to give Compressed PAF files containing read overlap information, GFA files describing the assembly graphs, FASTA files containing the assembled sequences ready for downstream genomic analysis tasks such as annotation, variant detection, or comparative genomics.
#------------------------------------------------------------------------------------------------------------------------------------------------#
#Unicycler
The below provided scripts are to run Unicycler, an assembly pipeline which combines the accuracy of short-read assembly with the contiguity of long-read assembly. It is particularly effective for assembling bacterial genomes, which are typically smaller and less complex than eukaryotic genomes. Unicycler uses both short-read data (typically Illumina) for accuracy and long-read data (Nanopore) to bridge gaps that are typically problematic for short-read assemblers.

#Unicycler short reads (merged R1 and R2)

#!/bin/bash

#SBATCH --job-name=unicycler_short
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=48:00:00
#SBATCH --output=/shared/home/payya4/slurm-%x-%j.out                  #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-%x-%j.err                  #replace "payya4" with your username
         

# Activate conda environment
source $HOME/.bash_profile
conda activate /shared/conda/shared


#Assembly with unicycler (short)
unicycler -t 8 \
-1 /workhere/students_2023/group6/merged_R1_short.fastq.gz \
-2 /workhere/students_2023/group6/merged_R2_short.fastq.gz \
-o /workhere/students_2023/group6/unicycler_short_result


# Deactivate conda environment
conda deactivate

#Expected result: This will contain the final assembly in various formats including FASTA files of the assembled contigs, a GFA file of the assembly graph, and logs detailing the assembly process.
#------------------------------------------------------------------------------------------------------------------------------------------------#

#unicycler long reads

#!/bin/bash

#SBATCH --job-name=unicycler_long
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=48:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out                  #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err                  #replace "mbxjk6" with your username

# Activate conda environment
source $HOME/.bash_profile
conda activate /shared/conda/shared

# Assembly with Unicycler (long - pass)
unicycler -t 8 \
-l /workhere/students_2023/group6/merged_long_reads_pass.fastq.gz \
-o /workhere/students_2023/group6/unicycler_long_pass


# Deactivate conda environment
conda deactivate

#expected result: files similar to the short-reads output, containing the long-read assembly results.
#------------------------------------------------------------------------------------------------------------------------------------------------#

##Unicycler (hybrid assembly)

#Unicycler hybrid pass

#!/bin/bash


#SBATCH --job-name=uni_hyb_p
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=48:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out                   #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err                  #replace "mbxjk6" with your username


echo "the job name is: $JOB_NAME_ID"

source $HOME/.bash_profile

#activate conda env
conda activate /shared/conda/shared


#run unicycler hybrid assembly (pass only)
unicycler -1 /workhere/students_2023/group6/merged_R1_short.fastq.gz \
-2 /workhere/students_2023/group6/merged_R2_short.fastq.gz \
-l /workhere/students_2023/group6/merged_long_reads_pass.fastq.gz \
-t 8 -o /workhere/students_2023/group6/unicycler_results/unicycler_hybrid_pass


#check unicycler.log tail to see if assembly has completed
#deactivte conda env
conda deactivate

#expected result: will both merged short-read and long-read data to create a highly accurate and contiguous assembly.
#------------------------------------------------------------------------------------------------------------------------------------------------#

#Unicycler hybrid pass and fail

#!/bin/bash


#SBATCH --job-name=uni_hyb_pf_2
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=72:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out                  #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err                  #replace "mbxjk6" with your username


echo "the job name is: $JOB_NAME_ID"

source $HOME/.bash_profile

#activate conda env
conda activate /shared/conda/shared

#run unicyler hybrid assembly (pass+fail)
unicycler -1 /workhere/students_2023/group6/merged_R1_short.fastq.gz \
-2 /workhere/students_2023/group6/merged_R2_short.fastq.gz \
-l /workhere/students_2023/group6/merged_long_reads_pf.fastq.gz \
-t 8 -o /workhere/students_2023/group6/unicycler_results/unicycler_hybrid_pf

#check unicycler.log tail to see if assembly has completed
#deactivte conda env
conda deactivate

#expected output: this will both "pass" and "fail" long-read data, potentially adding more depth and coverage but also introducing more noise.
#------------------------------------------------------------------------------------------------------------------------------------------------#
#No code for bandage
#------------------------------------------------------------------------------------------------------------------------------------------------#

##Quast (quality assesement of the assembly)
#The script here are designed to conduct quality assessment of  genomic assemblies using QUAST (Quality Assessment Tool for Genome Assemblies), which evaluates genome assemblies by computing various metrics, such as the number of contigs, the total length of the genome, N50, L50, and others. QUAST can also align assembled sequences to a reference genome if provided, which offers a deeper understanding of assembly quality, such as misassemblies, mismatches, and indels.

#installation
conda install -c bioconda quast
#recommanded installtion using sbatch

#QUAST for unicycler assemblies

#!/bin/bash

#SBATCH --job-name=quast_analysis
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00         
#SBATCH --output=/shared/home/payya4/slurm-quast-%x-%j.out         #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-quast-%x-%j.err         #replace "payya4" with your username


# Activate conda environment with QUAST installed
source $HOME/.bash_profile
conda activate /shared/conda/shared

# QUAST for short-read assembly
python /shared/conda/shared/bin/quast \
-o /workhere/students_2023/group6/quast_results/quast_short.unicycler \
-r /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/GCF_033550185.1_ASM3355018v1_genomic.fna \
-g /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/genomic.gff \
/workhere/students_2023/group6/unicycler_results/unicycler_short_result/assembly.fasta

# QUAST for long-read assembly (pass and fail)
python /shared/conda/shared/bin/quast \
-o /workhere/students_2023/group6/quast_results/quast_long_pf_unicycler \
-r /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/GCF_033550185.1_ASM3355018v1_genomic.fna \
-g /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/genomic.gff \
/workhere/students_2023/group6/unicycler_results/unicycler_long_result/assembly.fasta

# QUAST for long-read assembly (pass)
python /shared/conda/shared/bin/quast \
-o /workhere/students_2023/group6/quast_results/quast_long_pass_unicycler \
-r /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/GCF_033550185.1_ASM3355018v1_genomic.fna \
-g /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/genomic.gff \
/workhere/students_2023/group6/unicycler_results/unicycler_long_pass/assembly.fasta


# QUAST for hybrid assembly (short and long_pf)
#python /shared/conda/shared/bin/quast \
#-o /workhere/students_2023/group6/quast_results/quast_hybrid_pf \
#-r /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/GCF_033550185.1_ASM3355018v1_genomic.fna \
#-g /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/genomic.gff \
#/workhere/students_2023/group6/unicycler_results/unicycler_hybrid_pf/ADD MISSING PATHWAY

# QUAST for hybrid assembly (short and long_pass)
#python /shared/conda/shared/bin/quast \
#-o /workhere/students_2023/group6/quast_results/quast_hybrid_pass \
#-r /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/GCF_033550185.1_ASM3355018v1_genomic.fna \
#-g /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/genomic.gff \
#/workhere/students_2023/group6/unicycler_results/unicycler_hybrid_pass/ADD MISSING PATHWAY


#QUAST for minimap assemblies

# QUAST for minimap long-read (pass and fail)
python /shared/conda/shared/bin/quast \
-o /workhere/students_2023/group6/quast_results/quast_minimap_pf \
-r /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/GCF_033550185.1_ASM3355018v1_genomic.fna \
-g /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/genomic.gff \
/workhere/students_2023/group6/minimap_results/minimap_long_pf/minimap_long_pf.fasta



# QUAST for minimap long-read (pass)
python /shared/conda/shared/bin/quast \
-o /workhere/students_2023/group6/quast_results/quast_minimap_pass \
-r /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/GCF_033550185.1_ASM3355018v1_genomic.fna \
-g /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/genomic.gff \
/workhere/students_2023/group6/minimap_results/minimap_long_pass/minimap_long_pass.fasta



# Deactivate conda environment
conda deactivate
#------------------------------------------------------------------------------------------------------------------------------------------------#

##Quast final

#!/bin/bash


#SBATCH --job-name=quast_final_final
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-quast-%x-%j.out                   #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-quast-%x-%j.err                   #replace "mbxjk6" with your username


# Activate conda environment with QUAST installed
source $HOME/.bash_profile
conda activate /shared/conda/shared



# QUAST for all unicycler (short, long, hybrid) and minimap (long)
python /shared/conda/shared/bin/quast -o /workhere/students_2023/group6/quast_results/quast_final_final \
-r /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/GCF_033550185.1_ASM3355018v1_genomic.fna \
-g /workhere/students_2023/group6/ncbi_dataset/ncbi_dataset/data/GCF_033550185.1/genomic.gff \
/workhere/students_2023/group6/minimap_results/minimap_long_pass/minimap_long_pass.fasta \
/workhere/students_2023/group6/minimap_results/minimap_long_pf/minimap_long_pf.fasta \
/workhere/students_2023/group6/unicycler_results/unicycler_short_result/assembly.fasta \
/workhere/students_2023/group6/unicycler_results/unicycler_long_result/assembly.fasta \
/workhere/students_2023/group6/unicycler_results/unicycler_long_pass/assembly.fasta \
/workhere/students_2023/group6/unicycler_results/unicycler_hybrid_pass/assembly.fasta \
/workhere/students_2023/group6/unicycler_results/unicycler_hybrid_pf/assembly.fasta



# Deactivate conda environment
conda deactivate

#Expected output: Report Files: QUAST generates several important output files, including:report.txt (text summary of the statistics),report.tsv (tab-separated version, useful for downstream analysis), report.pdf (visual report containing plots and graphs), contigs_reports/ (detailed per-contig reports), aligned_stats/ (statistics on the alignment against a reference, if provided), Graphical Outputs: Depending on the flags and options set, QUAST can produce several plots, such as: Assembly histograms, GC content plots, Nx/Lx curves, Comprehensive Metrics: Key metrics provided by QUAST include: N50 and L50: Measures that help to understand the contiguity of the assembly. Total Length: Total size of all contigs, which can be compared against the expected genome size. Number of Contigs: Total number of sequences in the assembly. Number of Misassemblies: Errors in assembly as compared to the reference. Quality Scores: For mismatches, indels, and other errors relative to the reference. 
#------------------------------------------------------------------------------------------------------------------------------------------------#
##Busco (for analysis)
The scripts written are to perform BUSCO (Benchmarking Universal Single-Copy Orthologs) analysis on genomic assemblies. These analyses are crucial for evaluating the quality of these assemblies by checking the completeness in terms of expected conserved orthologous genes. The scripts are organized for hybrid assemblies (both successful and failed), short-read assemblies, long-read assemblies, and assemblies got from Minimap2 alignments.

#installation
conda install -c bioconda busco
#recommanded installtion using sbatch

#hybrid assemblies 

#!/bin/bash

#SBATCH --job-name=busco_analysis_2
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-busco-%x-%j.out                  #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-busco-%x-%j.err                  #replace "payya4" with your username

# Activate conda environment with BUSCO installed
source $HOME/.bash_profile
conda activate /shared/conda/busco

# BUSCO for assembly hybrid pass and fail
busco \
-i /workhere/students_2023/group6/unicycler_results/unicycler_hybrid_pf/assembly.fasta -o bsc_hyb_pf --out_path /workhere/students_2023/group6/busco_results_hyb -l archaea_odb10 -m genome

# BUSCO for assembly hybrid pass
busco \
-i /workhere/students_2023/group6/unicycler_results/unicycler_hybrid_pass/assembly.fasta -o bsc_hyb_pass --out_path /workhere/students_2023/group6/busco_results_hyb -l archaea_odb10 -m genome

# Deactivate conda environment
conda deactivate

#------------------------------------------------------------------------------------------------------------------------------------------------#

##Busco analysis

#!/bin/bash

#SBATCH --job-name=busco_analysis
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-busco-%x-%j.out                  #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-busco-%x-%j.err                  #replace "payya4" with your username

# Activate conda environment with BUSCO installed
source $HOME/.bash_profile
conda activate /shared/conda/busco

# BUSCO for short-read assembly
busco \
-i /workhere/students_2023/group6/unicycler_results/unicycler_short_result/assembly.fasta -o /workhere/students_2023/group6/busco_results/bsc_shortreads -l /shared/conda/busco_downloads/lineages/archaea_$


# BUSCO for long-read assembly (pass and fail)
busco \
-i /workhere/students_2023/group6/unicycler_results/unicycler_long_result/assembly.fasta -o /workhere/students_2023/group6/busco_results/bsc_longreads_pf -l /shared/conda/busco_downloads/lineages/archaea$

# BUSCO for long-read assembly (pass)
busco \
-i /workhere/students_2023/group6/unicycler_results/unicycler_long_pass/assembly.fasta -o /workhere/students_2023/group6/busco_results/bsc_longreads_pass -l /shared/conda/busco_downloads/lineages/archaea$

#BUSCO minimap

#BUSCO for minimap long-read (pass and fail)
busco \
-i /workhere/students_2023/group6/minimap_results/minimap_long_pf/minimap_long_pf.fasta -o /workhere/students_2023/group6/busco_results/mm_long_pf -l /shared/conda/busco_downloads/lineages/archaea_odb10 $

#BUSCO for minimap long-read (pass)
busco \
-i /workhere/students_2023/group6/minimap_results/minimap_long_pass/minimap_long_pass.fasta -o /workhere/students_2023/group6/busco_results/mm_long_pass -l /shared/conda/busco_downloads/lineages/archaea_$


# Deactivate conda environment
conda deactivate

#Expected output: short_summary.txt: A concise summary of the analysis, providing quick insights into the quality of the assembly in terms of gene content completeness. full_table.tsv: A detailed list of each BUSCO assessment, showing which specific genes were found complete, fragmented, or missing. missing_busco_list.tsv and duplicated_busco_list.tsv: Lists of missing and duplicated genes, if any. Graphical outputs: Often, BUSCO will generate plots that visually represent the completeness of the assembly, which are helpful for presentations or reports.

#------------------------------------------------------------------------------------------------------------------------------------------------#
# We got new set of samples of genomes. We repeated same procedure with these samples. The additional thing added were flye, prokka and genovi.
#new reads (barcodes)

barcode 01 and 02
#!/bin/bash

#SBATCH --job-name=nanoplot_01_02
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out                  #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err                  #replace "mbxjk6" with your username



source $HOME/.bash_profile


#activate conda env
conda activate /shared/conda/shared


#run nanoplot on barcode 01
NanoPlot --fastq /workhere/students_2023/group6/new_reads/raw_reads/barcode01/*fastq \
--threads 8 \
--plots kde dot \
-o /workhere/students_2023/group6/new_reads/nanoplot_results/barcode01_nanoplot


#run nanoplot on barcode02
NanoPlot --fastq /workhere/students_2023/group6/new_reads/raw_reads/barcode02_/* \
--threads 8 \
--plots kde dot \
-o /workhere/students_2023/group6/new_reads/nanoplot_results/barcode02_nanoplot


#deactivte conda
conda deactivate

#------------------------------------------------------------------------------------------------------------------------------------------------#

barcode 06

#!/bin/bash


#SBATCH --job-name=nanoplot_b6
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/payya4/slurm-%x-%j.out	   #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-%x-%j.err         #replace "payya4" with your username

source $HOME/.bash_profile

#activate conda env
conda activate /shared/conda/shared

#run nanoplot on long reads (pass)
NanoPlot --fastq /workhere/students_2023/group6/new_reads/raw_reads/barcode06/* \
         --threads 8 \
         --plots kde dot \
         -o /workhere/students_2023/group6/new_reads/nanoplot_results/barcode6_nanoplot

#deactivte conda env
conda deactivate

#------------------------------------------------------------------------------------------------------------------------------------------------#
#barcode 09

#!/bin/bash


#SBATCH --job-name=nanoplot_b9
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/payya4/slurm-%x-%j.out	    #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-%x-%j.err        #replace "payya4" with your username


source $HOME/.bash_profile

#activate conda env
conda activate /shared/conda/shared

#run nanoplot on long reads (pass)
NanoPlot --fastq /workhere/students_2023/group6/new_reads/raw_reads/barcode09/* \
         --threads 8 \
         --plots kde dot \
         -o /workhere/students_2023/group6/new_reads/nanoplot_results/barcode09_nanoplot

#deactivte conda env
conda deactivate

#------------------------------------------------------------------------------------------------------------------------------------------------#
#barcode 10

#!/bin/bash


#SBATCH --job-name=nanoplot_b10
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/payya4/slurm-%x-%j.out	   #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-%x-%j.err       #replace "payya4" with your username


source $HOME/.bash_profile

#activate conda env
conda activate /shared/conda/shared

#run nanoplot on long reads (pass)
NanoPlot --fastq /workhere/students_2023/group6/new_reads/raw_reads/barcode10/* \
         --threads 8 \
         --plots kde dot \
         -o /workhere/students_2023/group6/new_reads/nanoplot_results/barcode10_nanoplot

#deactivte conda env
conda deactivate

#------------------------------------------------------------------------------------------------------------------------------------------------#
##Merging reads

#!/bin/bash

#SBATCH --job-name=merged_reads
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=8g
#SBATCH --time=02:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out     #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err     # #replace "mbxjk6" with your username


# Merge barcode01 reads 
cat /workhere/students_2023/group6/raw_reads/raw_reads/barcode01/* > \
/workhere/students_2023/group6/raw_reads/merged_barcode01.fastq.gz 

# Merge barcode02 reads 
cat /workhere/students_2023/group6/raw_reads/raw_reads/barcode02/* > \
/workhere/students_2023/group6/raw_reads/merged_barcode02.fastq.gz  


# Merge barcode06 reads 
cat /workhere/students_2023/group6/raw_reads/raw_reads/barcode06/* > \
/workhere/students_2023/group6/raw_reads/merged_barcode06_pass.fastq.gz


# Merge barcode09 reads 
cat /workhere/students_2023/group6/raw_reads/raw_reads/barcode09/* > \
/workhere/students_2023/group6/merged_barcode09_pass.fastq.gz


# Merge barcode10 
cat /workhere/students_2023/group6/raw_reads/raw_reads/barcode10/* /workhere/students_2023/group6/fastq_fails_long/* > \
/workhere/students_2023/group6/merged_barcode10_pass.fastq.gz

#------------------------------------------------------------------------------------------------------------------------------------------------#

Barcodes reads assembly with unicycler

#!/bin/bash


#SBATCH --job-name=unicycler_barcodes_pass_reads
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=48:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out         #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err         #replace "mbxjk6" with your username


echo "the job name is: $JOB_NAME_ID"

# Activate conda environment
source $HOME/.bash_profile
conda activate /shared/conda/shared



# Assembly with Unicycler barcode09 (pass)
unicycler -t 8 \
-l /workhere/students_2023/group6/new_reads/merged_barcode09_pass.fastq.gz \
-o /workhere/students_2023/group6/new_reads/unicycler_results/barcode09_pass

# Assembly with Unicycler barcode10 (pass)
unicycler -t 8 \
-l /workhere/students_2023/group6/new_reads/merged_barcode10_pass.fastq.gz \
-o /workhere/students_2023/group6/new_reads/unicycler_results/barcode10_pass

# Assembly with Unicycler barcode06 (pass)
unicycler -t 8 \
-l /workhere/students_2023/group6/new_reads/merged_barcode06_pass.fastq.gz \
-o /workhere/students_2023/group6/new_reads/unicycler_results/barcode06_pass



# Deactivate conda environment
conda deactivate


#run from home directory
# sbatch /workhere/students_2023/group6/new_reads/unicycler.sh

#------------------------------------------------------------------------------------------------------------------------------------------------#

#!/bin/bash


#SBATCH --job-name=unicycler_02
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=48:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-%x-%j.out                   #replace "mbxjk6" with your username         
#SBATCH --error=/shared/home/mbxjk6/slurm-%x-%j.err                   #replace "mbxjk6" with your username
                  

echo "the job name is: $JOB_NAME_ID"

# Activate conda environment
source $HOME/.bash_profile
conda activate /shared/conda/shared



# Assembly with Unicycler barcode01 (pass)
#unicycler -t 8 \
#-l /workhere/students_2023/group6/new_reads/merged_barcode01.fastq.gz \
#-o /workhere/students_2023/group6/new_reads/unicycler_results/barcode01_pass

# Assembly with Unicycler barcode02 (pass)
unicycler -t 8 \
-l /workhere/students_2023/group6/new_reads/merged_barcode02.fastq.gz \
-o /workhere/students_2023/group6/new_reads/unicycler_results/barcode02_pass



# Deactivate conda environment
conda deactivate
#------------------------------------------------------------------------------------------------------------------------------------------------#
#QUAST Analysis on new reads (barcode)
#!/bin/bash


#SBATCH --job-name=quast_6_10
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-quast-%x-%j.out         #replace "payya4" with your username         
#SBATCH --error=/shared/home/payya4/slurm-quast-%x-%j.err          #replace "payya4" with your username     


# Activate conda environment with QUAST installed
source $HOME/.bash_profile
conda activate /shared/conda/shared



# QUAST (barcode 06, barcode10)
python /shared/conda/shared/bin/quast \
-o /workhere/students_2023/group6/new_reads/quast_results/quast_6_10 \
-r /workhere/students_2023/group6/new_reads/ncbi_dataset_2/ncbi_dataset/data/GCF_005406325.1/GCF_005406325.1_ASM540632v1_genomic.fna \
-g /workhere/students_2023/group6/new_reads/ncbi_dataset_2/ncbi_dataset/data/GCF_005406325.1/genomic.gff \
/workhere/students_2023/group6/new_reads/unicycler_results/barcode06_pass/assembly.fasta \
/workhere/students_2023/group6/new_reads/unicycler_results/barcode10_pass/assembly.fasta \



# Deactivate conda environment
conda deactivate

#------------------------------------------------------------------------------------------------------------------------------------------------#
#!/bin/bash


#SBATCH --job-name=quast_9
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-quast-%x-%j.out                   #replace "payya4" with your username         
#SBATCH --error=/shared/home/payya4/slurm-quast-%x-%j.err                   #replace "payya4" with your username         


# Activate conda environment with QUAST installed
source $HOME/.bash_profile
conda activate /shared/conda/shared



# QUAST (barcode 09)
python /shared/conda/shared/bin/quast \
-o /workhere/students_2023/group6/new_reads/quast_results/quast_9 \
-r /workhere/students_2023/group6/new_reads/Haloferax_volcanii/GCF_000025685.1/GCF_000025685.1_ASM2568v1_genomic.fna \
-g /workhere/students_2023/group6/new_reads/Haloferax_volcanii/GCF_000025685.1/genomic.gff \
/workhere/students_2023/group6/new_reads/flye_result/barcode09_pass/assembly.fasta \




# Deactivate conda environment
conda deactivate

#------------------------------------------------------------------------------------------------------------------------------------------------#
Busco Analysis for new reads (barcode)
#!/bin/bash

#SBATCH --job-name=busco_new
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-quast-%x-%j.out                   #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-quast-%x-%j.err                   #replace "payya4" with your username

# Activate conda environment with BUSCO installed
source $HOME/.bash_profile
conda activate /shared/conda/busco

# BUSCO for b06
busco \
-i /workhere/students_2023/group6/new_reads/unicycler_results/barcode06_pass/assembly.fasta \
-o /workhere/students_2023/group6/new_reads/busco_results/barcode06_busco \
-l haloferacales_odb10 \
-m genome




# BUSCO for b10
busco \
-i /workhere/students_2023/group6/new_reads/unicycler_results/barcode10_pass/assembly.fasta \
-o /workhere/students_2023/group6/new_reads/busco_results/barcode10_busco \
-l haloferacales_odb10 \
-m genome



# BUSCO for b09
busco \
-i /workhere/students_2023/group6/new_reads/flye_result/barcode09_pass/assembly.fasta \
-o /workhere/students_2023/group6/new_reads/busco_results/barcode09_busco \
-l haloferacales_odb10 \
-m genome




# Deactivate conda environment
conda deactivate

#------------------------------------------------------------------------------------------------------------------------------------------------#
#The scripts wriiten is for genome assembly using Flye, which is a de novo assembler for single-molecule sequencing reads from technologies Nanopore and PacBio. Flye specializes in creating high-quality assemblies from long reads, making it particularly suitable for complex genome projects. 

#installation
conda install -c bioconda flye
#recommanded installtion using sbatch


#for sample 01 and 02

#!/bin/bash

#SBATCH --job-name=flye_01_02
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/mbxjk6/slurm-quast-%x-%j.out                  #replace "mbxjk6" with your username
#SBATCH --error=/shared/home/mbxjk6/slurm-quast-%x-%j.err                  #replace "mbxjk6" with your username


echo "the job name is: $JOB_NAME_ID"

# Activate conda environment
source $HOME/.bash_profile
conda activate /shared/conda/shared


#flye assembly for barcode 01
flye --nano-raw /workhere/students_2023/group6/new_reads/merged_barcode01.fastq.gz \
--out-dir /workhere/students_2023/group6/new_reads/flye_result/barcode_01 \
--threads 8

flye --nano-raw /workhere/students_2023/group6/new_reads/merged_barcode02.fastq.gz \
--out-dir /workhere/students_2023/group6/new_reads/flye_result/barcode_02 \
--threads 8


#deactivte conda env
conda deactivate

#run from home directory
#sbatch /workhere/students_2023/group6/new_reads/flye01_02
#------------------------------------------------------------------------------------------------------------------------------------------------#
#for sample 09

#!/bin/bash

#SBATCH --job-name=flye_b9
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-quast-%x-%j.out                  #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-quast-%x-%j.err                  #replace "payya4" with your username

# Activate conda environment
source $HOME/.bash_profile
conda activate /shared/conda/shared

#flye for b9
flye --nano-raw /workhere/students_2023/group6/new_reads/merged_barcode09_pass.fastq.gz \
--out-dir /workhere/students_2023/group6/new_reads/flye_result/barcode09_pass \
--threads 8

#deactivte conda env
conda deactivate

#expected outcomes: Assembly Directories: Each specified output directory (flye_result/barcode_xx) will contain the Flye assembly outputs, which include: assembly.fasta: The primary output containing the assembled sequences. assembly_info.txt: Provides details about the contigs, including their lengths and whether they are circular. flye.log: Contains logs of the assembly process, useful for debugging and performance assessment.
#------------------------------------------------------------------------------------------------------------------------------------------------#
#The script written is for Prokka, a rapid prokaryotic genome annotation tool, to annotate genomic assemblies associated with different barcodes.Used specifically for barcode06, barcode10, and barcode09. 

#installation
conda install -c bioconda prokka

#recommanded installtion using sbatch

#!/bin/bash

#SBATCH --job-name=prokka_barcodes
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-quast-%x-%j.out                #replace "payya4" with your username
#SBATCH --error=/shared/home/payya4/slurm-quast-%x-%j.err                  #replace "payya4" with your username

# Activate conda environment
source $HOME/.bash_profile
conda activate /shared/conda/shared


#barcode 06
prokka --outdir /workhere/students_2023/group6/new_reads/prokka_results/barcode06_prokka \
--prefix prokka_b6 \
--kingdom archaea \
--genus haloferax \
/workhere/students_2023/group6/new_reads/unicycler_results/barcode06_pass/assembly.fasta


#barcode 10
prokka --outdir /workhere/students_2023/group6/new_reads/prokka_results/barcode10_prokka \
--prefix prokka_b10 \
--kingdom archaea \
--genus haloferax \
/workhere/students_2023/group6/new_reads/unicycler_results/barcode10_pass/assembly.fasta


#barcode 09
prokka --outdir /workhere/students_2023/group6/new_reads/prokka_results/barcode09_prokka \
--prefix prokka_b9 \
--kingdom archaea \
--genus haloferax \
/workhere/students_2023/group6/new_reads/flye_result/barcode09_pass/assembly.fasta

#deactivte conda env
conda deactivate

#expected outcome: For each barcode, Prokka will generate a set of files in the specified output directories, including: .gff: This file contains the Genome Feature Format annotations, including CDS, rRNA, tRNA, and other genomic elements, .gbk (GenBank file): A detailed annotation file that includes the genome sequence and its features in a format suitable for submission to GenBank, .fna: Nucleotide fasta of the annotated features, .faa: Protein fasta files of the CDS sequences, .ffn: Nucleotide fasta files of the CDS sequences, .sqn: An NCBI submission format file, .log: Contains log information from the annotation process.

#------------------------------------------------------------------------------------------------------------------------------------------------#
#The script written is to run Genovi, a visualization tool for genome assemblies and annotations. Genovi converts genome annotation files, typically in GenBank format, into visual representations that help researchers quickly assess the structure and content of their assemblies. This particular script runs Genovi on Prokka-generated GenBank files for different barcoded samples.

#installation
conda install -c bioconda genovi

#recommanded installtion using sbatch

#!/bin/bash

#SBATCH --job-name=genovi_all
#SBATCH --partition=hpc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=30G
#SBATCH --time=24:00:00
#SBATCH --output=/shared/home/payya4/slurm-quast-%x-%j.out
#SBATCH --error=/shared/home/payya4/slurm-quast-%x-%j.err

# Activate conda environment
source $HOME/.bash_profile
conda activate /shared/conda/genovi


# Change to the directory where genovi results should be written
cd /workhere/students_2023/group6/new_reads/genovi_results

# genovi for barcode06
genovi \
-i /workhere/students_2023/group6/new_reads/prokka_results/barcode06_prokka/prokka_b6.gbk \
-o genovi_6_result \
-s complete

# genovi for barcode09
genovi \
-i /workhere/students_2023/group6/new_reads/prokka_results/barcode09_prokka/prokka_b9.gbk \
-o genovi_9_result \
-s complete

# genovi for barcode10
genovi \
-i /workhere/students_2023/group6/new_reads/prokka_results/barcode10_prokka/prokka_b10.gbk \
-o genovi_10_result \
-s complete

# Deactivate conda environment
conda deactivate

#expected outcomes: Visual Reports: Genovi generates HTML-based reports that visually represent the genomic annotations. These reports are interactive and allow users to explore various genomic features, Output Directories: Each barcode's results are stored in their respective directories (genovi_6_result, genovi_9_result, genovi_10_result).
#------------------------------------------------------------------------------------------------------------------------------------------------#
