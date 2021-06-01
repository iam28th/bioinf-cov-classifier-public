# Sample classification

## Objectives
1. Identify a set of TCRs from high-throughput sequencing data (Shoukat et al.) that are associated with COVID-19 using MiXCR.
2. Identify and correct systematic error difference between Shoukat et al. and Shomuradova et al. repertoire sequencing datasets.
3. Develop classifier(s) that can tell COVID-19 status based on donor TCR repertoire.
4. Explore k-mer markers that allow to predict COVID-19 status - their location and possible origin.

## Methods

We used MiXCR to get annnotation and calculate clonotype frequency statistics from raw sequences data. Then V-usage normalization was applied to the generated feature tables and k-mer and VDJ-genes frequencied were calculated for each sample. The frequencies were used as input features for several machine learning models (namely, CatBoost ensemble and hierarchical clustering). We also performed statistical tests for each k-mer frequency and inspected location of the most significant ones. Python graphing libraries (matplotlib, seaborn and Logomaker) were used to visualize our findings.

<img src="plots/Flowchart.png" width="300">

## Results

V-usage normzliation noticeably reduces bias between datasets, as can be seen from PCA plots and hierarchical clustering results (unnormalized samples are on the left):

<p float="center">
  <img src="plots/Unnormalized 3-mers only.png " width="45%" />
  <img src="plots/V-usage normalization 3-mers only.png" width="45%" /> 
</p>

<p float="center">
  <img src="plots/cluster_by_origin_only_kmers_not_normalized.png" width="45%" />
  <img src="plots/cluster_by_origin_only_kmers.png" width="45%" /> 
</p>

We observed poor CatBoost performance when the train and test samples originated from different datasets:

<img src="plots/catboost_roc.png" width="900">

After applying statistical tests, we observed that top significant k-mers* (i.e., with the lowest p-value) are located towards the end of CDR3 sequence (left), which means they come from the J segment. They also allow for unambiguous decision boundry between Healthy and Convalescent groups (right) in the dataset.

<p float="center">
  <img src="plots/ssk_location_fixed.png" width="45%" />
  <img src="plots/ssk_pca_Shoukat.png" width="45%" /> 
</p>

Knowing that, we were able to identify a set of J segments that may distinguish Convalescent from Healthy*.

\* no significant k-mers or J segments were discovered in one of the datasets; poor consistency may be a result of systematic biases between TCR repertoire profiling methods (as discussed in [this study](https://www.nature.com/articles/s41587-020-0656-3)) or because of high individual diversity.

## Data availability
Apart from the [dataset](https://www.ebi.ac.uk/ena/browser/view/PRJEB38339) from _Shoukat et al._ article, this project used some yet unpublished samples that are not to be shared at the moment. Generated frequency tables, however, are free for use (files `freq_table.csv` and `freq_table_normalized.csv`).

## System requirements 

The work was performed using HP Pavilion Notebook with the following parameters:
* OS - Ubuntu 20.04.1 LTS x86_64 
* CPU - Intel i5-8300H (8) @ 4.000GHz
* RAM - 7809MiB

The following software was used during the course of the project:
* MiXCR v3.0.13
* Python 3.8.5
* Jupyter notebook 6.1.4

Third-party python libraries:
* numpy 1.19.2
* pandas 1.1.3
* scipy 1.5.2
* seaborn 0.11.1
* matplotlib 3.4.1
* scikitplot 0.3.7
* logomaker 0.8 
* catboost 0.24.4

## References 

1. Dmitriy A. Bolotin, Stanislav Poslavsky, Igor Mitrophanov, Mikhail Shugay, Ilgar Z. Mamedov, Ekaterina V. Putintseva, and Dmitriy M. Chudakov. "MiXCR: software for comprehensive adaptive immunity profiling." Nature methods 12, no. 5 (2015): 380-381.
2. Barennes, P., Quiniou, V., Shugay, M. et al. "Benchmarking of T cell receptor repertoire profiling methods reveals large systematic biases." Nat Biotechnol 39, 236–245 (2021). https://doi.org/10.1038/s41587-020-0656-3
3. Liudmila Prokhorenkova, Gleb Gusev, Aleksandr Vorobev, Anna Veronika Dorogush, Andrey Gulin. "CatBoost: unbiased boosting with categorical features." NeurIPS, 2018
4. M. Saad Shoukat, Andrew D. Foers, Stephen Woodmansey, Shelley C.Evans, Anna Fowler, Elizabeth J. Soilleux. "Use of machine learning to identify a T cell response to SARS-CoV-2." Cell Reports Medicine, 2021
 
