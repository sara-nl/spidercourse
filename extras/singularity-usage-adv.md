## Using singularity containers


Your analysis can be run with software that was installed either by the software manager/sys admin of the system or yourself. What if you want to run 
the same analysis on another system(s)? Or you want to simply test some workflow on Spider but don't want to install the  necessary software from scratch or mess up an existing installation? This is where
containers can come in extremely handy. As you do not have admin rights on the system, we do not support Docker containers. But the good news is that we do support Singularity containers! Let us run the same E coli analysis but by importing software from a singularity  container.

Let us first inspect what version of singularity is available on the system

```sh
singularity version
```

In this example we will directly provide you a read-only singularity image and run the workflow. Please login to Spider again and check if the required software is available to you

```
fastqc -h
trimmomatic
```

This will throw errors which means that the software is not available to you. Let us now download the scripts to run jobs that will use the Singularity containers

```sh
cd $HOME
mkdir ecoli-analysis-container
cd ecoli-analysis-container
wget https://raw.githubusercontent.com/sara-nl/spidercourse/master/scripts/job-submit-variant-calling-singularity-adv.sh

wget https://raw.githubusercontent.com/sara-nl/spidercourse/master/scripts/run-variant-calling-singularity-adv.sh
```

Let us see how the scripts set the software environment. The job-submit-variant-calling-singularity-adv.sh script does not set the software environemnt here. So, let us inspect the script that runs the analysis

```sh
cat run-variant-calling-singularity-adv.sh

#!/bin/bash
set -x
set -e
ecolipath=$HOME/ecoli-analysis-container

mkdir -p data/ref_genome
cp /project/surfadvisors/Data/ecoli-analysis/data/ref_genome/ecoli_rel606.fasta data/ref_genome/
ls data/ref_genome

mkdir data/trimmed_fastq_small
cp /project/surfadvisors/Data/ecoli-analysis/data/trimmed_fastq_small/*fastq data/trimmed_fastq_small/
ls data/trimmed_fastq_small

mkdir results
cd $ecolipath/results

genome=$ecolipath/data/ref_genome/ecoli_rel606.fasta

singularity exec $ecolipath/elixir-singularity.sif bwa index $genome

mkdir -p sam bam bcf vcf

for fq1 in $ecolipath/data/trimmed_fastq_small/*_1.trim.sub.fastq
    do
    echo "working with file $fq1"

    base=$(basename $fq1 _1.trim.sub.fastq)
    echo "base name is $base"

    fq1=$ecolipath/data/trimmed_fastq_small/${base}_1.trim.sub.fastq
    fq2=$ecolipath/data/trimmed_fastq_small/${base}_2.trim.sub.fastq
    sam=$ecolipath/results/sam/${base}.aligned.sam
    bam=$ecolipath/results/bam/${base}.aligned.bam
    sorted_bam=$ecolipath/results/bam/${base}.aligned.sorted.bam
    raw_bcf=$ecolipath/results/bcf/${base}_raw.bcf
    variants=$ecolipath/results/bcf/${base}_variants.vcf
    final_variants=$ecolipath/results/vcf/${base}_final_variants.vcf 
    
    #Align reads to reference genome
    singularity exec $ecolipath/elixir-singularity.sif bwa mem $genome $fq1 $fq2 > $sam
    
    #Convert from sam to bam format
    singularity exec $ecolipath/elixir-singularity.sif samtools view -S -b $sam > $bam

    #Sort the bam files    
    singularity exec $ecolipath/elixir-singularity.sif samtools sort -o $sorted_bam $bam 
    
    #Calculate the read coverage of positions in the genome
    singularity exec $ecolipath/elixir-singularity.sif bcftools mpileup -O b -o $raw_bcf -f $genome $sorted_bam
    
    #Detect the single nucleotide polymorphisms (SNPs)
    singularity exec $ecolipath/elixir-singularity.sif bcftools call --ploidy 1 -m -v -o $variants $raw_bcf 
    
    #Filter the SNPs for the final output in VCF format
    singularity exec $ecolipath/elixir-singularity.sif vcfutils.pl varFilter $variants > $final_variants
   
done
 ```

> **_Food for brain:_**
>
> * How is the software environment set up in the above script?
> * Does it matter where the singularity image is located?
> * Can you install some new software in this container? 

```
#Copy the container to the location as expected in the script

cp /project/surfadvisors/Software/elixir-singularity.sif  $HOME/ecoli-analysis-container

#Make sure the path to store the results in the variant calling script does not already have the results when you run the job
sbatch --job-name=var-call-singularity -J 'var-call-singularity' --output=%x-%j.out job-submit-variant-calling-singularity-adv.sh
```

Did the analysis run successfully? To test the status of your job you can run the command

```sh
squeue -u $USER
```

If your job has finished you can check the output log for errors

```sh
cat var-call-tmpdir-jobID.out #replace the jobID

#Another check would be the output of the following command
grep -v "#" $HOME/ecoli-analysis-container/results/vcf/SRR2589044_final_variants.vcf | wc -l

#The answer should be 10 (the number if expected variants detected in this population)
```

You got the same results as earlier examples with the container without setting a single path for your software! 

Wondering how the container was built? You can find the recipe [here](https://raw.githubusercontent.com/sara-nl/spidercourse/master/extras/singularity-recipe). This container was built on Singularity version 3.2.1-1 (on macOS Mojave running Vagrant). Using containers is simple but you should bear in mind that it should be properly built with all the required dependencies, and should be updated regularly and tested in the runtime environment. 
