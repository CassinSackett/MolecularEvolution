# Lab 1: Alignment

###### tags: `Unix`, `LONI`, `Multiple Sequence Alignment`

Note: This material is generated from the publicly available resources from the Woods Hole Molecular Evolution course. Credit goes to the instructors of that course.

Table of Contents
[ToC]


> The objective of this activity is to become familiar with the features of multiple alignment and alignment visualization programs. This includes data input and output, basic visualization and editing functions, and alignment options.
> By now, you should be able to move files between the remote machine and your own computer. Please see me if you are having trouble with this. 

### :arrow_right: About alignment
> Most analysis software assumes the input data (gene sequences, short reads, etc.) are already aligned. When using your own data, you will first need to conduct multiple sequence alignment.

### Optional background reading on alignment:
* [general background](https://molevolworkshop.github.io/labs/alignment/general-background/) from Woods Hole course
* background on [MAFFT](https://molevolworkshop.github.io/labs/alignment/MAFFT-background/) and [algorithms used](https://mafft.cbrc.jp/alignment/software/algorithms/algorithms.html) in MAFFT
* background on [MUSCLE](https://molevolworkshop.github.io/labs/alignment/MUSCLE-background/) and [user guide](https://drive5.com/muscle5/manual/commands.html)

### Preface on Using a Cluster
There are two ways to run computing jobs on QB3 and most other computing clusters: by initiating an interactive session or by submitting a batch job. During an interactive session, you can type commands and run programs on the cluster in real time. Your job will be killed if you log out of QB3 before your commands have finished executing. Interactive sessions are good for short analyses that require limited computing resources or for testing out new software. In contrast, a batch job allows you to submit an analysis to the cluster to be run in the future once the requested computing resources become available. All data and commands are preselected through script files and therefore run to completion without human contact. Batch jobs are good for complex analyses that run on many computing nodes for many hours or days.

## Today: Alignment in Muscle

We will use Muscle for sequence alignment, as its performance is comparable to MAFFT and it is faster. For amino acid alignments it seems to be more accurate (Pais et al. 2014). Muscle is described [here](https://academic.oup.com/nar/article/32/5/1792/2380623) and is loaded as a module on LONI (muscle/3.8.1551/intel-19.0.5). 

:fish: 

Today, we will use sequences that are already on LONI, and will download the alignment for visualization using [MEGA](https://megasoftware.net/) (or [[SeaView](https://doua.prabi.fr/software/seaview)](https://) or [AliView](http://www.ormbunkar.se/aliview/), if you prefer, although I will not provide instruction in those programs). If you do not already have MEGA downloaded, please [download it now](https://megasoftware.net/).

### Step 1: Get some sequences to align

You should always work from your /username/work/ directory. Files are deleted after 90 days, but there is no size limit on the directory, so it is perfect for this course (for files you don't want to lose, like scripts, you can copy them to your home directory where files are stored for longer). 

In your /work/ directory, create a new, empty directory named *MSAlab* and use the following command to copy the sequence files there:
```
cp /project/sackettl/MolEvol/data/sciuridae_genes/*txt MSAlab/
```

The files currently have a .txt extension, but they are fasta files. 

If you prefer to work with a different dataset, you can transfer sequences to LONI via scp or FileZilla (or use available sequences), run Muscle remotely, and then transfer back to your laptop for visualization. If you are working with large sequences, there is a job script called R_read-seqs.sh in the /MolEvol/MSA/ directory that you can copy, edit, and use to download sequences directly from GenBank to LONI. This way, you are not required to download them to your laptop first. 


### Step 2: Basic functions in MEGA

Copy one of the .txt files (which is really a fasta file) from LONI to your laptop :computer: using scp. These files contain nucleotide sequences (accessed from GenBank) from genes hypothesized to play a role in hibernation, language, and visual acuity. Change the extension to .fa or .fasta and load the dataset into MEGA.

Take a look at the data. Do the sequences look as though they have already been aligned? Try some of the basic commands. To select a taxon, click one one of the names on the left side. Explore the menus. Observe how sequences can be reversed, complemented, translated etc. Do not close the window, but move it aside for now.

### Step 3: Align sequences in MUSCLE
1. Copy the script /MolEvol/MSA/muscle_align.sh into your work directory. From here, you will edit the script for the sequences you want to align.
2. The skeleton of a MUSCLE command looks like this:
```
muscle -log muscle_gene1.log -in gene1.fasta -out muscle_gene1.fasta
```
3. The details of the command are:
    'muscle' starts the program MUSCLE
    '-log muscle_gene1.log' tells MUSCLE to put all the output except the alignment itself into a log file that includes things like the gap penalty used, etc.
    '-in gene1.fasta' specifies the input file to MUSCLE (in version 5, this is replaced by '-align')
    '-out muscle_gene1.fasta' tells MUSCLE to put the alignment in a file you named '*musclegene1.fasta*'.
4. We also have to tell LONI how to run this script--How much memory will it take? How much time? To whom are we charging the usage? These are specified in the first section of the script with the #SBATCH commands. You will need to edit the output (-o) and error (-e) files, as well as the time (-t) and memory (-n) needed for your particular alignment. In this class, you should always "charge" your computing hours to the allocation (-A) called "loni_molec_evo".
5. Finally, you also want to get rid of the extra stuff (here, namely, alignment of genes you do not care about). If you are deleting the steps that align other genes (i.e., if you align only one instead of four), or if you decide to run an alignment of long DNA sequences, adjust the compute time accordingly. Please do not over-charge the computing time! We are all sharing it.


### Step 4: Evaluate the MUSCLE alignment
Once the alignment is done, scp the aligned fasta file to your own computer and open it in MEGA. 

Look at the alignment. How many columns are there? Is the alignment as you expected, or is there anything that seems unusual? What would you expect an alignment to look like in protein coding versus non-protein coding genes?

Go to the main window and click on Phylogeny > Construct neighbor-joining tree.
![](https://i.imgur.com/opmZVhJ.png)

* Use a J-C distance model
* In the Gaps/missing data section, select pairwise deletion or partial deletion
* Calculate 100 bootstraps

:eyes: These trees are easy for helping to evaluate your alignments, but this program should never be your tree building method. :eyes: 

Compare the trees from the alignments of two different genes. Do the topologies or branch lengths differ? Was a single gene enough to resolve the phylogeny (i.e., no polytomies)?

Explore the sidebar to the left of the tree. Do all clades have bootstrap support? Branch lengths? Add these values, if they are not already there. What else can you manipulate?

:chipmunk:


### Extra fun - Step 5: Compare alignments performed using DNA versus protein sequences

See me if you need help getting started!



