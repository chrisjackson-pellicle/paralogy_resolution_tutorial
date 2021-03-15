# HybSeq Paralogy Resolution Tutorial
GAP tutorial for containerised Yang and Smith paralogy resolution pipeline 


https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4209138/

https://www.biorxiv.org/content/10.1101/2020.08.21.261925v2

https://bitbucket.org/dfmoralesb/target_enrichment_orthology/src/master/


This tutorial assumes that you have Singularity and Nextflow installed, and that you have the Y_and_S Singularity image downloaded. In addition, you should have the Nextflow pipeline script `xxx.nf` and its corresponding config file `xxx.config`.

Installation instructions for running on OSX via Vagrant here[LINK].

Instructions for customising the Nextflow config file for your computing resoruces here[LINK].


## Input data

### Paralog sequences

If you have used the Nextflow pipeline `hybpiper_pipeline_v1_6_NO_INTRONERATE.nf` to run HybPiper, it will have produced the following subfolders in your main `results` folder:

 - `09_paralogs`
 - `10_paralogs_noChimeras`

See the HybPiper tutorial[LINK] for a full description of the files in these output folders. Briefly, folder `09_paralogs` contains a fasta file for each gene in your HybPiper target file. Each fasta file contains the 'main' contig selected by HybPiper for each sample. Where HybPiper has detected putative paralog contigs, these sequences are also included; in such cases, the main contig has the fasta header suffix `.main`, whereas putative paralogs have the suffix `.0`, `.1` etc. Folder `10_paralogs_noChimeras` contains the same data, except putative chimeric contigs (see LINK) have been removed.        

***Tutorial step 1:***

    Copy folder `09_paralogs` into your current working directory (i.e. the directory containing `xxx.nf` and `xxx.config`.

### Outgroup sequences

Some of the paralogy resolution methods used in this pipeline require an outgroup sequence for each of your genes. These outgroup sequences can be provided in a fasta file (e.g. `outgroups.fasta`), with the same fasta header formatting as your HybPiper target file. For example, if you have used the Angiosperms353 target file, and you wish to use sequences from Sesame as your outgroup, your `outgroups.fasta` file might contain the following:

    >sesame-6995
    gtgggatatgaacaaaatccattgagcttgtattactgtta...
    >sesame-4757
    ctggtgcgtcgagcacttctcttgagcagcaacaatggcgg...
    >sesame-6933
    gaagtagatgctgtggtggtggaagcattcgacatatgcac...
    
    ...etc
    
Again, note that the gene identifier following the dash in the fasta headers (e.g. '6995' for header '>sesame-6995') needs to correspond to a gene identifier in your target file. 

It's fine if your `outgroups.fasta` file contains additional sequences. When running the pipeline (see below) you'll provide one or more taxon names using the parameter e.g. `--outgroups sesame`.

***Tutorial step 2:***

    Copy the fasta file containing outgroup sequences to your current working directory.


## Running the pipeline

### Nextflow pipeline command

***Tutorial step 3:***

    Run the pipeline using the command:
    
    nextflow_20_04 run alex_YS_pipeline_v1_6.nf -c nextflow_alex_YS.config -profile slurm -resume --hybpiper_paralogs_directory 06_paralogs --target_file Angiosperms353_targetSequences.fasta --outgroups Ambtr --pool 4 --threads 4

### Optional arguments
Optional arguments:

      -profile <profile>                              Configuration profile to use. Can use multiple (comma separated)
                                                      Available: standard (default), slurm
      --pool <int>                                    Number of threads for Python multiprocessing pool. 
                                                      Default is 1.
      --threads <int>                                 Number of threads per pool instance. Default is 1.
      --skip_process_01_hmmcleaner                    Skip the HmmCleaner step of process_01 (useful when HybPiper has been run with -nosupercontigs). 
      --no_supercontigs
      --process_02_trim_bad_ends_cutoff <int>         Default is 5.
      --process_02_trim_bad_ends_size <int>           Default is 15
      --skip_process_02_trim_bad_ends
      --process_04_trim_tips_relative_cutoff          Default is 0.2
      --process_04_trim_tips_absolute_cutoff          Default is 0.4
      --process_06_branch_length_cutoff               Default is 0.3
      --process_06_minimum_taxa                       Default is 3 
      --process_09_prune_paralog_MO_minimum_taxa      Default is 2
      --process_10_prune_paralogs_RT_minimum_ingroup_taxa
                                                      Default is 2  
      --process_11_prune_paralogs_MI_relative_tip_cutoff
                                                      Default is 0.2
      --process_11_prune_paralogs_MI_absolute_tip_cutoff
                                                      Default is 0.4
      --process_11_prune_paralogs_MI_minimim_taxa    
                                                      Default is 2


## Interpreting output data

After running the pipeline, output can be found in the folder `results` (unless you have changed the name of the default output folder using the `--outir <name>` parameter. This will consist of 23 subfolders, as described below.


**01_outgroup_added**

This folder contains your paralog fasta files, with outgroup sequences from your `outgroups.fasta` added. Fasta header for outgroup sequences will have the suffix `.outgroup`, e.g. `>sesame.outgroup`. 

**02_alignments**
Placeholder text.

**03_alignments_hmmcleaned**
Placeholder text.

**04_alignments_internalcut**
Placeholder text.

**05_tree_files**
Placeholder text.

**06_trim_tips**
Placeholder text.

**07_masked_tips**
Placeholder text.

**08_cut_internal_branches**
Placeholder text.

**09_selected_alignments**
Placeholder text.

**10_realigned**
Placeholder text.

**11_realigned_trees**
Placeholder text.

**12_prune_MO_trees**
Placeholder text.

**13_prune_RT_trees**
Placeholder text.

**14_prune_MI_trees**
Placeholder text.

**15_selected_alignments_MO**
Placeholder text.

**16_selected_alignments_RT**
Placeholder text.

**17_selected_alignments_MI**
Placeholder text.

**18_alignments_stripped_names_MO**
Placeholder text.

**19_alignments_stripped_names_MO_realigned**
Placeholder text.

**20_alignments_stripped_names_RT**
Placeholder text.

**21_alignments_stripped_names_RT_realigned**
Placeholder text.

**22_alignments_stripped_names_MI**
Placeholder text.

**23_alignments_stripped_names_MI_realigned**
Placeholder text.

**in_and_outgroups_list.txt**
Placeholder text.


## Post-pipeline analyses
Placeholder text.

### Pipeline parameters and options
Placeholder text.

### general notes
Placeholder text.

###################################





