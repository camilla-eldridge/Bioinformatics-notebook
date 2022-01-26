# Practical Notes 

Bookmark of tool commands and notes...

<br /> <br /> <br />

**Genome alignment**

nucmer alignment of two genomes in forward orientation:

            bsub -P teamID -n 24 -e nucmer_e -o nucmer_o -M 36000 -R"select[mem>36000] rusage[mem=36000] span[hosts=1]" /path/to/nucmer -f -p output_ID /path/to/genomeA.fa /path/to/genomeB.fa

<br /> <br /> <br />

**Dot** 

Dot plot of nucmer alignment above:

                        bsub -P teamID -e dot.e -o dot.o -M 4000 -R"select[mem>4000] rusage[mem=4000]" /path/to/DotPrep.py --delta output_ID.delta --out output_ID2

<br /> <br /> <br />

**Repeat library generation and masking** 

RepArk 
            ./RepARK.pl -l /path/to/genome.fna -o repeats_out
<br /> <br /> <br />

**RepeatMasker** 

            RepeatMasker genome.fna -lib repeat_lib.lib -xsmall -gff -nocut -noisy
<br /> <br /> <br />

**gushr**

            bsub -q normal -o gushr.log -n 15 -M20000 -R'select[mem>20000] rusage[mem=20000] span[hosts=1]' /path/to/gushr.py -t /path/to/gtf -b /path/to/bam -g /path/to/genome -o utr_output

<br /> <br /> <br />


# STAR
# Index genome, first round, second round...

    bsub -o index.txt -M32000 -R'select[mem>32000] rusage[mem=32000] span[hosts=1]'  -n 11 /path/to/star --runThreadN 8 --runMode genomeGenerate --genomeDir /path/to/genome_index_dir --genomeFastaFiles /path/to/genome

    bsub -o star.txt -M32000 -R'select[mem>32000] rusage[mem=32000] span[hosts=1]' -n 16 /path/to/star --genomeDir /path/to/genome_index_dir --runThreadN 16 --readFilesIn out.R1.fastq.gz,out.R2.fastq.gz --outFileNamePrefix out --outSAMtype BAM Unsorted --readFilesCommand zcat

    bsub -q basement -o star_round2.txt -M32000 -R'select[mem>32000] rusage[mem=32000] span[hosts=1]'  -n 16 /path/to/star --genomeDir /path/to/genome_index_dir --runThreadN 16 --readFilesIn out.R1.fastq.gz,out.R2.fastq.gz --outFileNamePrefix out2 --outSAMtype BAM Unsorted --readFilesCommand zcat --sjdbFileChrStartEnd outSJ.out.tab

#other options.....--outFilterMultimapNmax 1 
<br /> <br /> <br />

# VARUS

bsub -o varus_out -n 8 -q basement -M150000 -R"select[mem > 150000] rusage[mem=150000] span[hosts=1]" /path/to/runVARUS.pl --aligner=HISAT --readFromTable=1 --createindex=1 --logfile=varus.log
<br /> <br /> <br />

# WUBLAST
    bsub -o nrdb.o -e nrdb.e  -n 6 -M10000 -R"select[mem>10000] rusage[mem=10000] span[hosts=1]" /path/to/wublast/nrdb L1.fa L2.fa L3.fa -o L123_nrdb.fasta

<br /> <br /> <br />

# CD-Hit
bsub -o cdhit.o -e cdhit.e  -n 6 -M10000 -R"select[mem>10000] rusage[mem=10000] span[hosts=1]" /path/to//cd-hit-est -i /path/to/sequence.fa -c 1 -o L123_clusters

<br /> <br /> <br />

# BTK
            bsub -o out.txt -n 24 -M100000 -R"select[mem>100000] rusage[mem=100000] span[hosts=1]" -q long snakemake -p --use-conda --conda-prefix /path/to/blob_pipe/230420/.conda --directory /path/to/blobdir/ --configfile /path/to/config.yaml --latency-wait 100 --rerun-incomplete --stats ID.snakemake.stats -j 20 -s /path/to/insdc-pipeline/Snakefile --resources btk=1
<br /> <br /> <br />

# BUSCO with singularity
#for transcriptome
            bsub -q normal  -o busco.o -e busco.e -n 12 -M32000 -R"select[mem>32000] rusage[mem=32000] span[hosts=1]" /software/singularity-v3.6.4/bin/singularity                  exec --bind /bind/path: -e busco_5.1.2.sif  busco -m transcriptome -i /path/to/Trinity.fasta -l mollusca_odb10 -o out_ID -c 12 --offline                    --download_path busco_downloads -f --auto-lineage-euk

<br /> <br /> <br />

#for genome
             bsub -q normal -o busco.o -e busco.e -n 12 -M32000 -R"select[mem>32000] rusage[mem=32000] span[hosts=1]" /software/singularity-v3.6.4/bin/singularity                  exec --bind /bind/path: -e busco_5.1.2.sif  busco -m genome -i genome.fa -l arthropoda_odb10 -o genome -c 12 --offline --download_path busco_downloads -f

<br /> <br /> <br />

# samtools
             samtools fastq --reference reference_genome.fa -1 out.R1.fastq -2 out.R2.fastq input.cram

  if no reference given in the sam/bam then samtools can find it, it's embedded...
  otherwise specify the reference...

#To get multiple mapped reads
Get reads with flag 256 (not a primary alignment) using -f 256 (NB: -F excludes these reads) and exclude flag 4 (unmapped reads)
samtools view -@ 6 -b -f 256 -F 4 -h mybam.bwa-mem.bam > mybam.bwa-mem.flag256.bam

#To exclude all multi-mapped reads

             bsub -o uniq_map.txt  -n 6 -M10000 -R"select[mem>10000] rusage[mem=10000] span[hosts=1]" samtools view -h out2Aligned.out.bam | grep -v -e 'XA:Z:' -e 'SA:Z:' | samtools view -b > out2Aligned.unique_mapped.bam

#Check they are unique 
            samtools view output_filtered.bam | awk '{print $1}' | sort | uniq -d

<br /> <br /> <br />

# Trinity assembly
#singularity trinity run

            bsub -q basement  -o out.o -e error.e -n 12 -M32000 -R"select[mem>32000] rusage[mem=32000] span[hosts=1]" /software/singularity-v3.6.4/bin/singularity                  exec --bind /lustre:/lustre -e trinityrnaseq.v2.12.0.simg Trinity --seqType fq --left /path/to/4#2.cram.R1.fastq --right /path/to               /4#2.cram.R2.fastq  --max_memory 32G --CPU 12 --trimmomatic --SS_lib_type RF --bflyCPU 6 --bflyHeapSpaceMax 10G --trimmomatic --normalize_by_read_set --output    /path/to/trinity_out

<br /> <br /> <br />

# Some awks
#length of each sequence in mult fasta file

            cat genome.fa | awk '$0 ~ ">" {if (NR > 1) {print c;} c=0;printf substr($0,2,100) "\t"; } $0 !~ ">" {c+=length($0);} END { print c; }'


<br /> <br /> <br />


