# MAGNET
shell script pipeline for inferring ML gene trees for many loci (e.g. genomic SNP data)

LICENSE
-------

All code within the PIrANHA repository, including MAGNET pipeline code, is available "AS IS" under a generous GNU license. See the License.txt file for more information.

CITATION
-------

If you use scripts from this repository as part of your published research, then I require you to cite the PIrANHA repository and/or MAGNET package as follows (I will put everything in Zenodo and get doi's in the near future):

  Bagley, J.C. 2016. PIrANHA. GitHub repository, Available at: http://github.com/justincbagley/PIrANHA/.
  
  Bagley, J.C. 2016. MAGNET. GitHub package, Available at: http://github.com/justincbagley/PIrANHA/MAGNET.

Alternatively, please provide the following link to this software program in your manuscript:

  http://github.com/justincbagley/PIrANHA/MAGNET
  
Example citations using the above URL: 
	"We estimated a gene tree for each SNP locus in RAxML v8 (Stamatakis 2014) using 
	the MAGNET pipeline in the PIrANHA github repository (http://github.com/justincbagley/PIrANHA/MAGNET).
	Each RAxML run specified the GTRGAMMA model and coestimated the maximum-likelihood phylogeny
	and bootstrap proportions from 500 bootstrap pseudoreplicates."

INTRODUCTION
-------

The estimation of species-level phylogenies, or "species trees" is a fundamental goal in evolutionary biology. However, while "gene trees" estimated from different loci provide insight into the varying evolutionary histories of different parts of the genome, gene trees are random realizations of a stochastic evolutionary process. Thus, gene trees often exhibit conflicting topologies, being incongruent with each other and incongruent with the underlying species tree due to a variety of genetic and biological processes (e.g. gene flow, incomplete lineage sorting, introgression, selection). 

With the advent of recent advances in DNA sequencing technologies, biologists now commonly sequence data from multiple loci, and even hundreds to thousands of loci can quickly be sequenced using massively parallel sequencing on NGS sequencing platforms. Full-likelihood or Bayesian algorithms for inferring species trees and population-level parameters from multiple loci, such as *BEAST and SNAPP, are computationally burdensome and may be difficult to apply to large amounts of data or distantly related taxa (or other cases that complicate obtaining MCMC convergence). By contrast, a number of recently developed and widely used "summary-statistics" approaches rely on sets of gene trees to infer a species tree for a set of taxa (reviewed by Chifman and Kubatko, 2014; Mirarab and Warnow, 2015). These methods are specifically designed to estimate gene trees or use gene trees input by the user, which are treated as observed data points analyzed in a distance-based or coalescent algorithm. Moreover, summary-statistics approaches to species tree inference tend to be accurate and typically much faster than full-data approaches (e.g. Mirarab et al., 2014;  Chifman and Kubatko, 2014). Examples of species tree software in this category include programs such as BUCKy, STEM, spedeSTEM, NJst, AUSTRAL, and ASTRID. Phylogenetic network models implemented in recent software like SplitsTree and SNaQ also improve network and inference by analyzing sets of gene trees. 

Despite the importance of gene trees in species tree and network inference, few resources have been specifically designed to aid rapid estimation of gene trees for different loci. MAGNET (MAny GeNE Trees) is a shell script pipeline within the PIrANHA (PhylogenetIcs ANd PHylogeogrAphy) github repository (https://github.com/justincbagley/) that helps fill this gap by automating extracting individual loci from a multilocus sequence alignment file and inferring a maximum-likelihood (ML) gene tree for each locus. The MAGNET package was originally coded up to aid analyses of SNP loci generated by massively parallel sequencing of ddRAD-seq genomic libraries of freshwater fishes. However, MAGNET can be used to estimate gene trees for loci in other multilocus data types with the appropriate format using conversion scripts provided within the package (see below). 

HARDWARE AND SETUP
-------

MAGNET focuses on allowing users to automate the workflow necessary for quickly estimating many gene trees for many loci on their local machine. No special hardware or setup is necessary, unless the user is interested in estimating gene trees on a remote supercomputing cluster. In that case, the user is referred to the c-MAGNET or "cluster MAGNET" software repository (https://github.com/justincbagley/c-MAGNET/).


SOFTWARE DEPENDENCIES
-------

MAGNET relies on several software dependencies. These dependencies are described in some detail in README files for different scripts in the package; however, here I provide a list of them, with asterisk marks preceding those already included in the MAGNET distribution:

- Perl (available at: https://www.perl.org/get.html).
- *Nayoki Takebayashi's file conversion Perl scripts (available at: http://raven.iab.alaska.edu/~ntakebay/teaching/programming/perl-scripts/perl-scripts.html).
- Python (available at: ).
- bioscripts.convert v0.4 Python package (available at: https://pypi.python.org/pypi/bioscripts.convert/0.4; also see README for "NEXUS2gphocs.sh").
- RAxML, installed and running on remote supercomputer (available at: http://sco.h-its.org/exelixis/web/software/raxml/index.html).

Users must install all software not included in MAGNET, and ensure that it is available via the command line on their local machine. On the user's local machine, Perl should be available by simply typing "Perl" at the command line; Python should be available by simply  typing "python" at the command lnie; and bioscripts.convert package should be available by typing "convbioseq" at the command line. Also, RAxML should be compiled using SSE3 install commands, so that when you are logged into your supercomputer by ssh pipe, RAxML should be called by simply typing "raxmlHPC-SSE3". For detailed instructions for setting up RAxML this way, refer to the newest RAxML user manual (available at: http://sco.h-its.org/exelixis/resource/download/NewManual.pdf).

INPUT FILE FORMAT
-------

MAGNET assumes that you are starting from multilocus DNA sequence data in a single datafile in G-Phocs (Gronau et al. 2011) format, with the extension ".gphocs", or in NEXUS format with the extension ".nex". For genomic data such as RAD tags or other SNP data derived from genotyping-by-sequencing (GBS)-type methods, it is recommended that the user assemble the data, call SNPs, and output SNP data files in various formats including .gphocs format in pyRAD (Eaton REF) or ipyrad (Eaton REF) prior to using MAGNET. However, this may not always be possible, and .gphocs format is not yet among the most popular file formats in phylogenomics/population genomics. Thus, I have added a "NEXUS2gphocs.sh" shell script utility within MAGNET that will convert a sequential NEXUS file into .gphocs format for you. 

PIPELINE
-------

Apart from input file conversion steps, the MAGNET pipeline works by calling three different scripts, in series, each designed to conduct a task that yields output that is processed in the next step of the pipeline. In STEP #1, the "gphocs2multiPhylip.sh" shell script is used to extract loci from the input file and place each locus in a Phylip-formatted file with extension ".phy". In STEP #2, a shell script named "MultiRAxMLPrepper.sh" is used to place the .phy files into separate folders ("run folders"), and prepare them to be run in RAxML. In STEP #3, a script named "RAxMLRunner.sh" is called to run RAxML on the contents of each run folder. In a "clean-up" step, MAGNET moves all .phy files files remaining in the working directory after STEP #3 to a new folder, "phylip_files", that is created in the working directory.

After running the MAGNET pipeline, the shell script "getGeneTrees.sh" automates post-processing of the output, including organizing all inferred gene trees into a single "gene_trees" folder in the working directory, and combining the individual 'best' gene trees resulting from each run into a single file named "besttrees.tre".




August 29, 2016
Justin C. Bagley, Brasília, DF, Brazil
