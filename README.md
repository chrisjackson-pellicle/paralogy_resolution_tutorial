# HybSeq Paralogy Resolution Tutorial
GAP tutorial for containerised Yang and Smith paralogy resolution pipeline 



Link to the [Yang and Smith 2014 manuscript][4]

https://www.biorxiv.org/content/10.1101/2020.08.21.261925v2

https://bitbucket.org/dfmoralesb/target_enrichment_orthology/src/master/


This tutorial assumes that you have Singularity and Nextflow installed, and that you have the Y_and_S Singularity image downloaded. In addition, you should have the Nextflow pipeline script `yang_and_smith_pipeline_v1_7.nf` and its corresponding config file `yang_and_smith.config`.

Installation instructions for running on OSX via Vagrant here[LINK].

Instructions for customising the Nextflow config file for your computing resoruces here[LINK].


## Input data

### Paralog sequences

If you have used the Nextflow pipeline `hybpiper_pipeline_v1_7.nf` to run HybPiper, it will have produced the following subfolders in your main `results` folder:

 - `11_paralogs`
 - `12_paralogs_noChimeras`

See the HybPiper tutorial[LINK] for a full description of the files in these output folders. Briefly, folder `11_paralogs` contains a fasta file for each gene in your HybPiper target file. Each fasta file contains the 'main' contig selected by HybPiper for each sample. Where HybPiper has detected putative paralog contigs, these sequences are also included; in such cases, the main contig has the fasta header suffix `.main`, whereas putative paralogs have the suffix `.0`, `.1` etc. Folder `12_paralogs_noChimeras` contains the same data, except putative chimeric contigs (see LINK) have been removed.        

***Tutorial step 1:***

    Copy folder `11_paralogs` into your current working directory (i.e. the directory containing `yang_and_smith_pipeline_v1_7.nf` and `yang_and_smith.config`.

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

It's fine if your `outgroups.fasta` file contains additional sequences. When running the pipeline (see below) you'll provide one or more taxon names using the parameter e.g. `--outgroups sesame`. You can provide more than one outgroup taxon name using an comma-separated list, e.g. `--outgroups sesame,taxon2,taxon3` etc.

***Tutorial step 2:***

    Copy the fasta file containing outgroup sequences to your current working directory.


## Running the pipeline

### Nextflow pipeline command

See section [Pipeline parameters and options](#pipeline-parameters-and-options) for a full explanation of available parameters and flags. The required parameters are:

    --hybpiper_paralogs_directory <directory>    Path to folder containing HybPiper paralog fasta files)
    --outgroups_file <file>                      File containing fasta sequences of target genes
    --outgroups <taxon1,taxon2,taxon3...>        A comma-separated list of outgroup taxa to add, in order of 
                                                 preference

***Tutorial step 3:***

    Run the pipeline using the command:
    
    nextflow run yang_and_smith_pipeline_v1_7.nf -c yang_and_smith.config -profile slurm -resume --hybpiper_paralogs_directory 06_paralogs --outgroups_file outgroups.fasta --outgroups sesame --pool 4 --threads 4

## Interpreting output data

After running the pipeline, output can be found in the folder `results` (unless you have changed the name of the default output folder using the `--outir <name>` parameter. This will consist of 23 subfolders, as described below.


- **`01_outgroup_added`**

  This folder contains your paralog fasta files, with outgroup sequences from your `outgroups.fasta` added. Fasta header for outgroup sequences will have the suffix `.outgroup`, e.g. `>sesame.outgroup`. 

- **`02_alignments`**

  Contains fasta files of untrimmed and trimmed alignments for your paralog fasta files and outgroups. Trimming is performed using [trimal][1] using the settings `-gapthreshold 0.12 -terminalonly -gw 1`. 

- **`03_alignments_hmmcleaned`**

  Contains fasta files of trimmed alignments that have been cleaned using [HmmCleaner][2] with default settings.

- **`04_alignments_internalcut`**

  Contains fasta files of HmmCleaned alignments that have undergone the following processes:

  - The 5' and 3' termini of paralog sequences are trimmed to the outgroup sequences. 
  - Paralog sequences are examined for internal gaps larger than 15 bases (default, user configurable), and sequence either side of the gap are trimmed until they match the given outgroup sequence [CJJ: change this to allow a different sequence, i.e. a more closely related target sequence] for at least five bases (default, user configurable).

  Note that this step can be skipped by using the `--skip_process_02_trim_bad_ends` flag. 

- **`05_tree_files`**

  Contains tree files in newick format, derived from trimmed and QC'd alignment. Trees are generated using [IQTree][3] with the settings `-m GTR+G -bb 1000 -bnni`.

- **`06_trim_tips`**

  Contains tree files with long tips pruned out via the following processes:

  - Trim tips that > relative_cutoff and >10 times longer than sister branch 
  - Trim any tips that are > absolute_cutoff

  Default values for relative_cutoff and absolute_cutoff are 0.2 and 0.4, respectively. These values will be dataset specific and can be altered using the parameters `--process_04_trim_tips_relative_cutoff <float>` and `--process_04_trim_tips_absolute_cutoff <float>`.

- **`07_masked_tips`**

  Contains pruned tree files where close alleles from same sample have been removed. This process is intended to remove all but one of multiple terminals from the same sample, i.e. not deep paralogues but two alleles or paralogues that are very close together. The tip that has the most unambiguous, well-aligned characters in the trimmed alignment is kept.

- **`08_cut_internal_branches`**

  Contains tree files with deep paralogs removed, by cutting long internal branches above a given length (default is 0.3, user configurable with the `--process_06_branch_length_cutoff <float>` parameter. Only trees with a minimum number of taxa after pruning are retained (default value 3, user configurable with the parameter `--process_06_minimum_taxa <int>`. 

- **`09_selected_alignments`**

  Contains fasta files of alignments of sequences present in the final, QC'd trees from the previous step.

- **`10_realigned`**

  Contains fasta files of re-aligned, re-trimmed sequences from the previous step.

- **`11_realigned_trees`**

  Contains tree files in newick format, derived from re-aligned, re-trimmed sequences from the previous step. Trees are generated using [IQTree][3] with the settings `-m GTR+G -bb 1000 -bnni`.

- **`12_prune_MO_trees`**

  Contains tree files for tree pruned using the MO method (see the [Yang and Smith 2014 manuscript][4], Figure 1 for an explanation).

- **`13_prune_RT_trees`**

  Contains tree files for tree pruned using the RT method (see the [Yang and Smith 2014 manuscript][4], Figure 1 for an explanation).

- **`14_prune_MI_trees`**

  Contains tree files for tree pruned using the MI method (see the [Yang and Smith 2014 manuscript][4], Figure 1 for an explanation).

- **`15_selected_alignments_MO`**

  Contains fasta files of alignments corresponding to sequences present in tree files output by the MO method.

- **`16_selected_alignments_RT`**

  Contains fasta files of alignments corresponding to sequences present in tree files output by the RT method.

- **`17_selected_alignments_MI`**

  Contains fasta files of alignments corresponding to sequences present in tree files output by the MI method.

- **`18_alignments_stripped_names_MO`**

  Contains fasta files of alignments corresponding to sequences present in tree files output by the MO method, with sequences names stripped and suitable for concatenation.

- **`19_alignments_stripped_names_MO_realigned`**

  Contains fasta files of re-aligned sequences from MO alignements from the previous step.

- **`20_alignments_stripped_names_RT`**

  Contains fasta files of alignments corresponding to sequences present in tree files output by the RT method, with sequences names stripped and suitable for concatenation.

- **`21_alignments_stripped_names_RT_realigned`**

  Contains fasta files of re-aligned sequences from RT alignements from the previous step.

- **`22_alignments_stripped_names_MI`**

  Contains fasta files of alignments corresponding to sequences present in tree files output by the MI method, with sequences names stripped and suitable for concatenation.

- **`23_alignments_stripped_names_MI_realigned`**

  Contains fasta files of re-aligned sequences from RT alignements from the previous step.

- **`in_and_outgroups_list.txt`**

  A text file containing a list of designated ingroup (`IN` in column 1) or outgroup (`OUT` in column 1) taxa, used by some of the paralogy resolution methods. 


## Post-pipeline analyses

e.g.

- Concanated IQtree
- Astral

etc.

### Pipeline parameters and options

Mandatory arguments:

      --hybpiper_paralogs_directory <directory>    Path to folder containing HybPiper paralog fasta files.
      --outgroups_file <file>                      File containing fasta sequences of outgroup sequences for each gene.
      --outgroups <taxon1,taxon2,taxon3...>        A comma-separated list of outgroup taxa to add, in order of 
                                                   preference.  
Optional arguments:

      -profile <profile>                              Configuration profile to use. Can use multiple (comma separated)
                                                      Available: standard (default), slurm
      --pool <int>                                    Number of threads for the Python multiprocessing pool. Used in e.g. alignments and tree-building steps. 
                                                      Default is 1, so e.g. one alignment will be run at a time during alignment steps.
      --threads <int>                                 Number of threads per multiprocessing pool. Used for programs that support multi-threading (e.g. mafft,
                                                      IQTree). Default is 1.
      --no_supercontigs                               Use this flag if you are processing paralogs from a run of HybPiper that used the --nosupercontigs flag. Mafft alignments with re-aligned using clustal omega, which can do a better job in these cases. Default is off.
      --process_02_trim_bad_ends_cutoff <int>         Number of bases either side of an internal gap that much match the reference before trimming stops.Default is 5.
      --process_02_trim_bad_ends_size <int>           Number of continuous internal gap positions for an internal gap to be investigated. Default is 15.
      --skip_process_02_trim_bad_ends                 Skips the step trimming the ends of internal gaps.
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

### General notes

e.g.

- Manually reviewing trees from a preliminary run to select appropriate cut-off values for tree pruning

etc.





[1]: http://trimal.cgenomics.org/ "Link to trimal website"
[2]: https://bmcecolevol.biomedcentral.com/articles/10.1186/s12862-019-1350-2 "Link to HmmCleaner manuscript"
[3]: http://www.iqtree.org/ "Link to IQtree website"
[4]: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4209138/ "Link to the Yang and Smith 2014 manuscript"

