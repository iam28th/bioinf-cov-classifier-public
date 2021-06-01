# Epitope classification

## Goal
The goal is to use several TCR specificity classifiers, such as TCRdist and GLIPH to tell A*02 YLQ and A*02 RLQ specific TCRs (those are coronavirus epitopes) from other virus-specific TCRs.

## Objectives

1. Study two algorithms for TCR classification, tcrdist3 (Mayer-Blackwell et al. 2020) and GLIPH2 (Huang et al. 2020).
2. Filter the initial database for our purposes (see Methods)
3. Apply both algorithms to the resulting data
4. Estimate the efficiency of each algorithm in terms of multi-class classification: make ROC curves, precision-recall curves and confusion matrices.
5. Estimate the efficiency of each algorithm in terms of binary classification: same as above.

## Methods

We considered two algorithms for TCR classification, namely tcrdist3 (https://www.biorxiv.org/content/10.1101/2020.12.24.424260v1) and GLIPH2 (https://pubmed.ncbi.nlm.nih.gov/32341563/). Data were preprocessed in the way that was needed for each algorithm and used for further analysis (see Description and Usage). VDJdb database (Bagaev et al. 2019) was used as the raw data. We filtered the database to retain only epitopes starting with YLQ (these are coronavirus epitopes), GLC, GIL and NLV (Influenza A, EBV and CMV epitopes, respectively) of the MHC A02 allele without references from 10X Genomics as 10X datasets are too large. Only the beta chain was used for the analysis. We used epitopes starting with GLC, GIL and NLV as a control set in binary classification task.

### tcrdist3
For tcrdist3, Python package of the same name was used. Distances between TCR amino acid sequences were calculated using the `TCRrep` function with the following parameters: `organism = 'human', chains = 'beta'`. Then, we used the function `get_basic_centroids` to get the clusters of TCRs and clonotypes constituting each cluster. 

### GLIPH2
For GLIPH2, web interface available at http://50.255.35.37:8080 was used with the following parameters: Algorithm - GLIPH2, Reference version - 1.0, Reference - CD8, all_aa_interchangeable - YES. The resulting file was downloaded and used for downstream analysis.
**NB:**  since GLIPH2 can assign one TCR to multiple clusters, it was decided to mark such sequences as unclassified. Moreover, the algorithm was able to classify only a small amount of TCRs (see Results), therefore, its usability is questioned for now. We estimated classification efficiency with the following assumptions:
1. Clusters with <10 members were filtered out.
2. Unclassified clonotypes were included in the analysis, marked as "Unclassified".
3. Then, clonotypes that were assigned to multiple clusters were filtered out.

For each algorithm, we came up with the following workflow:
![Workflow](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/workflow.png)

For each cluster, distribution of TCRs (and the corresponding epitopes) was calculated. After that, epitope with the most share in cluster was assigned as a predicted epitope of this cluster. These weights were then used as a probabilities for calculation of ROC curves, precision-recall curves and confusion matrices. All aforementioned estimations were computed for multi-class classification task (performance of the algorithms in telling epitopes specific to SARS-CoV-2, Influenza A, EBV and CMV from each other) and binary classification task (performance of the algorithms in telling SARS-CoV-2-specific epitopes from other epitopes). 

## Results
### tcrdist3

For multi-class classification, sensitivity and specificity were sufficient as well as precision and recall. However, confusion matrices showed a decent prevalence in classifying epitopes as EBV or CMV epitopes, with high accuracy only for the CMV class. For GLC and YLQ classes, the performance was not satisfying at all since accuracy for these classes appeared to be only 1%. Overall accuracy of multi-class classification for tcrdist3 of 54.3% was not sufficient. Moreover, weighted accuracy at this step was only 27.75%. However, this could happen because of imbalance of the classes in the initial dataframe (e.g., only ~300 coronavirus epitopes in the VDJdb after the filtration).  

![multi_tcrdist_curves](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/tcrdist3/tcrdist3_multiclass_curves.png)
![multi_tcrdist3_cm](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/tcrdist3/tcrdist3_multiclass_confusion_matrix.png)

The situation was even worse upon binary classification. Despite the fact that ROC curve has changed a little, there is a significant drop of the precision-recall curve, suggesting decrease of the respective metrics. After looking at the confusion matrix, we observed a class accuracy of <1% in case of coronavirus epitopes, which makes tcrdist3 not suitable for the task of epitope prediction.

![binary_tcrdist3_curves](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/tcrdist3/tcrdist3_binary_curves.png)
![binary_tcrdist3_cm](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/tcrdist3/tcrdist3_binary_confusion_matrix.png)

### GLIPH2

Overall performance of GLIPH2 was better in both multi-class and binary classification cases.

![multi_gliph2_curves](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/gliph2/gliph2_multiclass_curves.png)
![multi_gliph2_cm](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/gliph2/gliph2_multiclass_confusion_matrix.png)

Due to the peculiarities of the algorithm (see Methods), only 11% of all clonotypes in the data were classified. Nevertheless, overall performance of the samples that were classified was better than in case of tcrdist3 (see above). At the same time, most of the clonotypes were not assigned with a cluster at all or were assigned to multiple clusters. This caused a lot of technical difficulties in estimmating efficiency of this algorithm and that is why confusion matrix for multi-class classification, as well as ROC curves and PRC curves, were made only for the part of the data that were classified by GLIPH2.

![binary_gliph2_curves](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/gliph2/gliph2_binary_curves.png)
![binary_gliph2_cm](https://github.com/iam28th/bioinf-cov-classifier-public/blob/master/epitope-classifier/gliph2/gliph2_binary_confusion_matrix.png)

Class accuracy of the epitopes starting with YLQ is not satisfying as well (22% class accuracy, 61% weighted accuracy in case of binary classification), albeit better than in case of tcrdist3. 

In conclusion, the estimation of GLIPH2 was very controversial and full of surprises. Due to the specifics of the algorithm, it is hard to recommend using it or not.

## Description and Usage

This part of the project consists of several scripts and files to be briefly described below.

* `00_custom_functions.R`: definition of the function to plot confusion matrix. **Arguments**: `df.true` - dataframe with true values, `df.pred` - dataframe with predicted values, `title` - main title of the resulting plot, `true.lab` - label of the true class, `pred.lab` - label of the predicted class, `high.col` and `low.col` define color scale for the plot.
* `01_data_preprocessing.R`: script for preprocessing VDJdb. Wow! The input is path to the folder with the database (change `path` variable in the script if you want to use it). After processing the file (the database is in this repo under the name `vdjdb.slim.txt`), two .tsv files (filtered database formatted as input for tcrdist3 and GLIPH2, respectively) are produced and stored in the folders of the same names.
* `02_gliph2.R`: analysis of the GLIPH2 output. Produces plots with ROC curves, precision-recall curves and confusion matrices for multi-label classification and binary classification. See the script for more details.
* `03_tcrdist3.R`: analysis of the tcrdist3 output. Produces plots with ROC curves, precision-recall curves and confusion matrices for multi-label classification and binary classification. See the script for more details.
* `04_benchmark.R`: main script that launches previously mentioned scripts one by one for convenience.
* `tcrdist3_clustering.py`: computing TCR distances and members of each cluster for tcrdist3. **Arguments**: `-i` - path to the input file, `-o` - path to the output file, `--chain` - chain to be used for computations. See `python3 tcrdist3_clustering.py -h` for details.

In addition, two folders under the names of used algorithms are present in the repo. They contain the input files used for the analysis, the resulting output files and resulting plots and metrics.

## Data availability

VDJdb is available and stored in this repository under the name `vdjdb.slim.txt` or at https://vdjdb.cdr3.net.
tcrdist3 is described in this [article](https://www.biorxiv.org/content/10.1101/2020.12.24.424260v1) (Mayer-Blackwell et al. 2020), documentation available [here](https://tcrdist3.readthedocs.io).
GLIPH2 is described in this [article](https://pubmed.ncbi.nlm.nih.gov/32341563/)  (Huang et al. 2020) and available at http://50.255.35.37:8080.

## System requirements

The commands and examples mentioned in this README have been tested on x86_64 Ubuntu 20.04 LTS with Intel(R) Core(TM) i7-3630QM CPU, 8 Gb system memory with the following software:

* R v3.6.3
* Python v3.8.5

Third-party R libraries:
* `data.table` v1.13.0
* `tidyr` v1.1.2
* `dplyr` v1.0.2
* `ggplot2` v3.3.2
* `patchwork` v1.1.0.9
* `pROC` v1.17.0.1

Third-party Python packages:
* `pandas` v1.2.4
* `argparse` v3.2
* `tcrdist3` v0.2.0

Databases:
* `VDJdb` (Bagaev et al. 2019)

## References

1. Koshlan Mayer-Blackwell, Stefan Schattgen, Liel Cohen-Lavi, Jeremy Chase Crawford, Aisha Souquette, Jessica A. Gaevert, Tomer Hertz, Paul G. Thomas, Philip Bradley, Andrew Fiore-Gartland. TCR meta-clonotypes for biomarker discovery with tcrdist3: quantification of public, HLA-restricted TCR biomarkers of SARS-CoV-2 infection. bioRxiv 2020.12.24.424260; doi: https://doi.org/10.1101/2020.12.24.424260

2. Huang H, Wang C, Rubelt F, Scriba TJ, Davis MM. Analyzing the Mycobacterium tuberculosis immune response by T-cell receptor clustering with GLIPH2 and genome-wide antigen screening. Nat Biotechnol. 2020 Oct;38(10):1194-1202. doi: 10.1038/s41587-020-0505-4. Epub 2020 Apr 27. PMID: 32341563; PMCID: PMC7541396.

3. Dmitry V Bagaev, Renske M A Vroomans, Jerome Samir, Ulrik Stervbo, Cristina Rius, Garry Dolton, Alexander Greenshields-Watson, Meriem Attaf, Evgeny S Egorov, Ivan V Zvyagin, Nina Babel, David K Cole, Andrew J Godkin, Andrew K Sewell, Can Kesmir, Dmitriy M Chudakov, Fabio Luciani, Mikhail Shugay. VDJdb in 2019: database extension, new analysis infrastructure and a T-cell receptor motif compendium. Nucleic Acids Research, Volume 48, Issue D1, 08 January 2020, Pages D1057–D1062, https://doi.org/10.1093/nar/gkz874
