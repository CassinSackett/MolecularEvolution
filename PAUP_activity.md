# Lab 3: Model Selection in PAUP

###### tags: `PAUP*`, `model selection`

> Note: This material is generated from the publicly available resources from the Woods Hole Molecular Evolution course. Credit goes to the instructors of that course. 
 
- Table of Contents 
[ToC]


## :cactus: Let's start with PAUP*!

### Step 1: Explore PAUP* command line and file formats

Refer to the PAUP* [quick start guide](http://paup.phylosolutions.com/tutorials/quick-start/) for more background.

:owl: 


### Step 2: Run PAUP* Interactively on LONI

We will use the command-line version of PAUP* installed in /MolEvol/software/. GUI versions are also available for Mac and Windows (but check versioning notes for available options) on the [PAUP* page](http://phylosolutions.com).

First, start an interactive session on LONI (refer back to the Alignment activity if you do not know what this means).
```
srun --time=1:00:00 --ntasks=8 --nodes=1 --account=loni_molec_evo --partition=single --pty /bin/bash
```

Take a look at the structure of the Nexus file primate-mtdna.nex in the data folder.
```
less /data/primate-mtdna.nex
```
Can you pick out what the different blocks of text are doing?

Next, launch PAUP* and load the primate mtdna Nexus file. From the /MolEvol/ directory:
```
software/paup4a168_ubuntu64 data/primate-mtDNA.nex -L testlog.txt
```

The -L flag creates a log file that will receive all output that is displayed on the screen during the run. Name this file something informative and put it in your personal directory.

**IMPORTANT: Searches in PAUP are extremely slow if model parameters are estimated during a tree search. It is almost always better to estimate model parameters on a fixed tree, and then fix those parameters prior to initiating the tree search.** The "automodel" command (see below) does this for you. However, if you estimate model parameters manually, you need to run the command ```lset fixall;``` to fix the model parameters to the estimated values.

Let's create a starting tree by computing a BioNJ tree using Jukes-Cantor distances:
```
dset distance=jc;
nj bionj=y;
```
We'll use the **automodel** command to perform model selection. To see what the available options and current settings are, you can run
```
automodel ?
```
These settings are fine for us for now, so let's start the run:
```
automodel;
```
Note that the output provides several metrics of model fit (e.g., BIC, AICc). The AICc criterion chooses the Tamura-Nei+Gamma (TrN+G) model for the primate-mt dataset, and this model is automatically set up in PAUP* for subsequent analysis, using the fixed parameter values just estimated.

We can now search for a maximum likelihood tree in PAUP*. First, we have to change the active optimality criterion to maximum likelihood. The default criterion is parsimony, not because the author thinks parsimony is a good method, but because it can be used on any kind of character data.
```
set criterion=likelihood;
```
Initiate a heuristic search using the default search settings:
```
hsearch;
```
and examine the tree that was found:
```
describe/plot=phylogram;
```
A somewhat safer searching strategy is to use different starting points, similar to RAxML and other programs. The usual way to do this in PAUP* is to use "random addition sequences" to generate different starting trees by stepwise addition. For difficult datasets, many starting trees for branch swapping will be different, and the searches may land in different parts of the tree space. If all random-addition sequence starts end up finding the same tree, you can be reasonably confident that you have found the optimal tree.

Let's do a random-addition-sequence search using the current model settings:
```
hsearch addseq=random nreps=50;
```
Obviously, this is an easy dataset: most, if not all, starting points find the same tree.

PAUP* now has the ability to initiate searches using RAxML, Garli, and FastTree2. The current PAUP* model settings are used to determine the option settings for these programs. Input and/or config files are created, the program is invoked as an external command, and the resulting trees are imported back into PAUP*. For example, let's look at the options available for the raxml command in PAUP*:
```
raxml ?
```
Start a RAxML search after changing the ML settings to optimize all parameters:
```
lset estall;
raxml execute=raxml;
```
Now let's optimize the tree from RAxML in PAUP*:
```
lscores;
```
Note that the likelihood score from PAUP* is slightly worse than the one reported by RAxML. This is because PAUP* is using the Tamura-Nei model rather than the more general GTR model, which RAxML does not support. To validly compare likelihoods, we need to set up the model in PAUP* to match the one used by RAxML. This involves undoing the restriction on the GTR substitution rates imposed by Tamura-Nei:
```
lset rclass=(abcdef);
lscores;
```
The Tamura-Nei model had set rclass=(abaaca), so that all 4 of the transversions are occurring at the same rate, but each of the transitions are potentially occurring at distinct rates. After the change to 'rclass' PAUP* gets a slightly better likelihood than RAxML. Presumably, PAUP* just did a slightly better job at optimizing the model parameters and branch lengths.

:monkey:

## Partitioned Analyses
PAUP* can do "partitioned" analyses in which different models are assigned to different subsets of sites (e.g., genes, codon positions, or a combination of these). The easiest way to set up a partitioned analysis is to use the **autopartition** command.

Partitioning begins by defining a set of basic "blocks", which are the smallest divisions of sites. The goal is to see if an adequate model can be achieved by combining some of these blocks into larger ones, thereby reducing model unnecessary model complexity (and speeding searches because fewer parameters need to be estimated).

The ```primate-mtdna.nex``` data file defines a partition called ```codons``` that places sites into one of four categories: 1st, 2nd, and 3rd positions, plus non-protein-coding (tRNA) sites. Let's first look at the options:

```
autopartition ?
```

Now start an automated partitioned analysis (this may take a few minutes):
```
autopartition partition=codons;
```
Partitioning by BIC combines 1st position and noncoding blocks into the same larger subset. You can evaluate the likelihood of the resulting model with the lscores command:
```
lscores;
```

Before starting a search, you should fix all model parameters to the MLEs obtained above rather than optimizing parameters during the search (which is extremely slow!)
```
lset fixall;
lscores;
```
The second ```lscores``` command above verifies that the parameters have been fixed to the MLE values. Now you can do a search as before:
```
set criterion=likelihood;
hsearch;
describe/plot=phylogram;
```

Note that PAUP*'s autopartition capability was inspired by Rob Lanfear's [PartitionFinder](http://www.robertlanfear.com/partitionfinder/) program, which has a few extra features. There is a helpful [tutorial](http://www.robertlanfear.com/partitionfinder/tutorial/) on the program.

## PAUP* summary
The above is intended to just give a flavor of how PAUP* works. We've barely scratched the surface with respect to what it can do. PAUP* can perform parsimony analysis under a variety of cost schemes, neighbor-joining and criterion-based distance searches using a large number of distance transformations, exact tree searches for smaller numbers of taxa, consensus tree calculation, agreement subtrees, distances between trees, analysis of base frequency composition, likelihood under amino-acid models, likelihood for discrete character data, and much more.


:::info
More info here ➜ [PAUP* website](https://paup.phylosolutions.com/) 
:::

### Step 3: Write a PAUP* job script for LONI

With most real datasets, you will want to run your PAUP* script in batch rather than interactive mode. So, let's translate what we did above to a script that you can submit on LONI.

Start with your slurm commands that request a queue, walltime, etc. Be sure to name your output files clearly.

Copy the relevant lines of code from the previous section (but re-name the log file so it doesn't overwrite your previous file) and submit your script! 

:bat: 


