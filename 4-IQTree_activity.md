# Lab 3: IQ-Tree, or Where do turtles belong? :turtle:

###### tags: `likelihood`, `phylogenetics`, `tree-building`

> Note: This material is generated from the publicly available resources from the Woods Hole Molecular Evolution course. Credit goes to the instructors of that course.

- Table of Contents
[ToC]


> Note: IQ-Tree is installed as a conda environment in our /project/ directory. I named the environment 'IQt_env'.  
> :female-teacher: 
> We will use a subset of data presented in [Chiari et al. 2012](https://doi.org/10.1186/1741-7007-10-65). Those data are in our /project/ directory, but you also will want to download the [fasta file](http://www.iqtree.org/workshop/data/turtle.fa) to take a look at the aligned sequences on your own computer. 
> 
 

## :turtle: Let's get started! :turtle:

### Step 1: View the alignment on your computer

Open the alignment (fasta) file in MEGA and the [partition (nexus) file](http://www.iqtree.org/workshop/data/turtle.nex) in BBEdit/Notepad. 
:::success
**QUESTIONS**
* Can you identify the gene boundary from the viewer? Does it roughly match the partition file?
* Is there missing data? Which taxa seem to have the most missing data?
* How could missing data be problematic?

:::



:crocodile: 

### Step 2: Infer the first phylogeny

You can now start to reconstruct a maximum-likelihood (ML) tree for the turtle dataset. You will submit a job script from within your /user/work/ directory that invokes a conda environment in my /project/ directory, using data in our class folder:

```
#SBATCH parameters

source /project/sackettl/miniconda3/etc/profile.d/conda.sh
source activate IQt_env

iqtree -s /project/sackettl/MolEvol/data/turtle.fa -bb 1000 -nt AUTO
```

This simple command will perform three important steps:
1. Select the best-fit model using [ModelFinder](https://www.nature.com/articles/nmeth.4285)
2. Reconstruct the ML tree using the [IQ-TREE search algorithm](https://academic.oup.com/mbe/article/32/1/268/2925592)
3. Assess branch support using the ultrafast bootstrap [UFBoot](https://academic.oup.com/mbe/article/30/5/1188/997508)

Once the run is done, IQ-TREE will write several output files, including:

> * turtle.fa.iqtree: the main report file with the computation results and a textual representation of the final tree
>
> * turtle.fa.treefile: the ML tree in Newick format, which can be visualized in FigTree or any other tree viewer program.
>
> * turtle.fa.log: log file of the entire run
>
> * turtle.fa.ckp.gz: checkpoing file used to resume an interrupted analysis

:tanabata_tree: 

:::success
**QUESTIONS**
* Look at the report file turtle.fa.iqtree. What is the best-fit model? What do you know about this model?
* Visualize the tree turtle.fa.treefile in FigTree. Compare the tree with [the published tree](https://bmcbiol.biomedcentral.com/articles/10.1186/1741-7007-10-65). Are they the same or different? If different, where are the differences?
* Look at the bootstrap support values. Which branch(es) have a low support?

:::



### Step 3: Apply partition model

Let's perform a [partition model analysis](https://academic.oup.com/sysbio/article/65/6/997/2281634) where each partition can have its own model:
```
iqtree -s turtle.fa -spp turtle.nex -bb 1000 -nt AUTO
```
where the -spp turtle.nex part specifies an [edge-linked proportional partition model](https://academic.oup.com/sysbio/article/65/6/997/2281634). This means that there is only one set of branch lengths, but each partition can have proportionally shorter or longer tree length, representing slow or fast evolutionary rate, respectively.
* Look at the report file turtle.nex.iqtree. What are the lowest- and highest-evolving genes?
* Compare the AIC/AICc/BIC score of partition model versus un-partition model above. Which model is better?
* Visualize the tree turtle.nex.treefile in FigTree and compare it with the tree from the unpartitioned model. Are they the same or different? If different, where is the difference? Which tree agrees with [the published tree](https://bmcbiol.biomedcentral.com/articles/10.1186/1741-7007-10-65)?
* Look at the bootstrap support values. Which branches have low support?


### Step 4: Choose the best partitioning scheme

Let's perform the [PartitionFinder](https://academic.oup.com/mbe/article/29/6/1695/1000514) algorithm that tries to merge partitions to reduce the potential over-parameterization.
```
iqtree -s turtle.fa -spp turtle.nex -bb 1000 -nt AUTO -m MFP+MERGE -rcluster 10 -pre turtle.merge
```
where
> * -m MFP+MERGE performs PartitionFinder followed by tree reconstruction
> * -rcluster 10 reduces computations by examining only the top 10% of partitioning schemes using the [relaxed clustering algorithm](https://bmcecolevol.biomedcentral.com/articles/10.1186/1471-2148-14-82)
> * -pre turtle.merge sets the prefix for all output files to turtle.merge.* to avoid overwriting outputs from previous analyses

:::success
**QUESTIONS**
* Look at the report file turtle.merge.iqtree. How many partitions do we have now? 
* Look at the AIC/AICc/BIC scores. Are they better or worse than those of the un-partition and partition models we did previously?
* What does the tree look like now? How are the bootstrap supports?
:::


 ### Step 5: Tree topology tests
We now want to know whether the trees inferred for the turtle dataset have significantly different log likelihoods. We can do this with the SH test ([Shimodaira & Hasegawa 1999](https://academic.oup.com/mbe/article/16/8/1114/2925508)) or with expected likelihood weights ([Strimmer & Rambaut 2002](https://royalsocietypublishing.org/doi/10.1098/rspb.2001.1862)).

First, concatenate the trees constructed by single and partition models into one file:
```
cat turtle.fa.treefile turtle.nex.treefile > turtle.trees
```
Or, more generally
```
cat unpartitioned.treefile partitioned.treefile > all.trees
```

(for Windows computers, replace 'cat' with 'type')

Now, pass this file to IQ-TREE via the ```-z``` option:
```
iqtree2 -s turtle.fa -p yourfolder/turtle.nex.best_scheme.nex -z turtle.trees -n 0 -wpl --prefix turtle.test
```
where
* ```-p turtle.nex.best_scheme.nex``` provides the partition model found previously to avoid running ModelFinger again
* ```-z turtle.trees``` inputs a set of trees
* ```-n 0``` avoids tree search and just performs tree topology tests
* ```-wpl``` prints partition-wise log likelihoods for both trees
* ```--prefix turtle.test``` sets the prefix for all output files to turtle.text

:::success
Look at the report file ```turtle.test.iqtree```. There is a new section called USER TREES. Do the two trees have significantly different log-likelihoods?


:bulb: **Hint:** To assess whether models are significantly different using likelihood, we use a Likelihood Ratio Test (LRT) between the two models. If one model is significantly better than the other, the ratio will differ from 1. [See here](https://stephens999.github.io/fiveMinuteStats/likelihood_ratio_simple_models.html) for an example.

:bulb: **Hint 2:** The KH and SH tests return p-values. bp-RELL and c-ELW return posterior weights, which are **not** p-values. The weights sum to 1 across all trees tested.

:::

:lizard:


### Step 6: Concordance factors
So far, we have assumed that gene trees and species trees are equal. However, it is well known that gene trees may be discordant. Therefore, we now want to quantify the agreement between gene trees and the species tree using a *concordance factor* ([Minh et al. 2020](https://academic.oup.com/mbe/article/37/9/2727/5828940)). If the below command does not work, change ```iqtree``` to ```iqtree-beta```.

You first need to compute the gene trees, one for each partition separately:

```
iqtree -s turtle.fa -S turtle.nex -pre turtle.loci -nt 4
```
where
```-S turtle.nex``` tells IQ-TREE to infer separate trees for every partition in ```turtle.nex```. All output files are similar to a partition analysis, except that the tree ```turtle.loci.treefile``` now contains a set of gene trees.

:::info
**Definitions**:
* Gene concordance factor (gCF) is the percentage of *decisive* (parsimony informative) gene trees concordant with a particular branch of the species tree (0% <= gCF(b) <= 100%). gCF(b)=0% means that branch b does not occur in any gene trees, whereas gCF(b)=100% means that branch b occurs in every gene tree.
* Site concordance factor (sCF) is the percentage of decisive alignment sites supporting a particular branch of the species tree (~33% <= sCF(b) <=-100%). sCF<33% means that another discordant branch *b'* is more supported, whereas sCF=100% means that branch *b* is supported by all sites.
* **CAUTION** when gCF ~ 0% or sCF<33%, even if bootstrap supports are ~100%!
* **GREAT** when gCF and sCF > 50% (i.e., branch is supported by a majority of genes and sites).

:::

You can now compute gCF and sCF for the tree inferred under the partition model:

```
iqtree -t turtle.nex.treefile --gcf turtle.loci.treefile -s turtle.fa --scf 100
```
where
* ```-t turtle.nex.treefile``` specifies a species tree
* ```--gcf turtle.loci.treefile``` specifies a gene-trees file
* ```--scf 100``` draws 100 random quartets when computing sCF

Once finished, this run will write several files:
* ```turtle.nex.treefile.cf.tree``` is a tree file where branches are annotated with bootstrap/gCF/sCF values
* ```turtle.nex.treefile.cf.stat``` is a table file with various statistics for every branch of the tree

Similarly, you can compute gCF and sCF for the tree under unpartitioned model:
```
iqtree-beta -t turtle.fa.treefile --gcf turtle.loci.treefile -s turtle.fa --scf 100
```

:::success
QUESTIONS:
* Visualise ```turtle.nex.treefile.cf.tree``` in FigTree.
* How do gCF and sCF values look compared with bootstrap supports?
* Visualise ```turtle.fa.treefile.cf.tree```. How do these values look like now on the contradicting branch?
:::


### Step 7: Resampling partitions and sites

Instead of bootstrap resampling sites, it is recommended to resample partitions and then sites within resampled partitions (Hoang et al., 2018). This may help to reduce over-confident branch supports.

```
iqtree -s turtle.fa -spp turtle.nex -bb 1000 -nt AUTO -bsam GENESITE -pre turtle.bsam
```
where
* ```-bsam GENESITE``` turns on resampling partition and sites strategy.
* ```-pre turtle.bsam``` sets the prefix for all output files as turtle.bsam. This is to avoid overwriting outputs from the previous analysis.

:::success
**QUESTIONS**:
* Is there any change in the tree topology?
* Do the bootstrap support values get smaller or larger?
:::


### Step 8: Identifying most influential genes
Now we want to investigate the cause for such topological difference between trees inferred by single and partitioned models. One way is to identify genes contributing the most phylogenetic signal towards one tree but not the other.

How can we do this? We can look at the gene-wise log-likelihood differences between the two given trees, T1 and T2. Those genes having the largest logL(T1)-logL(T2) will be in favor of T1, whereas genes with the largest logL(T2)-logL(T1) favor T2.

With the ```-wpl``` option used above, IQ-TREE will write partition-wise log-likelihoods into the  ```turtle.test.partlh``` file.

:::success
**QUESTIONS**
* Import this file into Excel or another spreadsheet software. Compute the partition-wise log-likelihood differences between the two trees. 
* What are the two genes that most favor the tree inferred by the single model?
* Take a look at the paper by [Brown & Thomson (2016)](https://academic.oup.com/sysbio/article/66/4/517/2950896). Compare the two genes you found with those from this paper. What is special about these two genes?
:::

:::warning
**FINAL QUESTION**:
Given all analyses done in this tutorial, which tree do you think is the true tree? :turtle: :bird: :crocodile: 
:::

---

