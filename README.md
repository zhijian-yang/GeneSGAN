# Gene-SGAN
Gene-SGAN is a multi-view semi-supervised clustering method for disentangling disease heterogeneity. By jointly considering brain phenotypic and genetic data, Gene-SGAN identify disease subtypes with associated phenotypic and genetic signatures. Using healthy control (HC) populations as a referece distribution, the model effectively cluster participants based disease-related phenotypic variations with genetic associations, thus avoiding confounders from disease-unrelated factors.


![image info](./datasets/Smile-GAN.png)

## License
Copyright (c) 2016 University of Pennsylvania. All rights reserved. See[ https://www.cbica.upenn.edu/sbia/software/license.html](https://www.cbica.upenn.edu/sbia/software/license.html)

## Installation
We highly recommend the users to install **Anaconda3** on your machine. After installing Anaconda3, Smile-GAN can be used following this procedure:

We recommend the users to use the Conda virtual environment:

```bash
$ conda create --name genesgan python=3.8
```
Activate the virtual environment

```bash
$ conda activate genesgan
```
Install SmileGAN from PyPi:

```bash
$ pip install GeneSGAN
```



## Input structure
Main functions of GeneSGAN basically takes three panda dataframes as data inputs, **phenotype_data**, **gene_data**, and **covariate** (optional). Columns with name *'participant_id'* and *diagnosis* must exist in **phenotype_data** and **covariate**. Some conventions for the group label/diagnosis: -1 represents healthy control (HC) and 1 represents patient (PT); categorical variables, such as sex, should be encoded to numbers: Female for 0 and Male for 1, for example. 

Genetic features of all PT but not HC participants in the **phenotype_data** need to be provided, so **gene_data** should not have the column *diagnosis*.
The current package only takes SNP data as genetic features, and each SNP variant in **gene_data** need to be recoded into 0, 1, or 2 indicating the number of minor alleles. 

Example for **phenotype_data**:

```bash
participant_id    diagnosis    ROI1    ROI2 ...
subject-1	    -1         325.4   603.4
subject-2            1         260.5   580.3
subject-3           -1         326.5   623.4
subject-4            1         301.7   590.5
subject-5            1	       293.1   595.1
subject-6            1         287.8   608.9
```
Example for **gene_data**:

```bash
participant_id    SNP1    SNP2 ...
subject-1         0       1
subject-2         1       0
subject-3         1       0
subject-4         0       0
subject-5	      2       0
subject-6         0       2
```

Example for **covariate**

```bash
participant_id    diagnosis    age    sex ...
subject-1	    -1         57.3   0
subject-2 	     1         43.5   1
subject-3           -1         53.8   1
subject-4            1         56.0   0
subject-5            1	       60.0   1
subject-6            1         62.5   0
```

## Example
We offer a toy dataset in the folder of SmileGAN/dataset.

```bash
import pandas as pd
from SmileGAN.Smile_GAN_clustering import single_model_clustering, cross_validated_clustering, clustering_result

train_data = pd.read_csv('train_roi.csv')
covariate = pd.read_csv('train_cov.csv')

output_dir = "PATH_OUTPUT_DIR"
ncluster = 3
start_saving_epoch = 20000
max_epoch = 30000

## three parameters for stopping threshold
WD = 0.12
AQ = 25
cluster_loss = 0.01

## one parameter for consensus method
consensus_type = "highest_matching_clustering"
```

When using the package, ***WD***, ***AQ***, ***cluster\_loss***, ***batch\_size*** need to be chosen empirically:

***WD***: Wasserstein Distance measures the distance between generated PT data along each direction and real PT data. (**Recommended value**: 0.11-0.14)

***AQ***: Alteration Quantity measures the number of participants who change cluster labels during last three traninig epochs. Low AQ implies convergence of training. (**Recommended value**: 1/20 of PT sample size)

***cluster\_loss***: Cluster loss measures how well clustering function reconstruct sampled Z variable. (**Recommended value**: 0.01-0.015)

***batch\_size***: Size of the batch for each training epoch. (Default to be 25) It is **necessary** to be reset to 1/8 of the PT sample size.

Some other parameters, ***lam***, ***mu*** have default values but need to be changed in some cases:

***lam***: coefficient controlling the relative importance of cluster\_loss in the training objective function. (Default to be 9) 

***mu***: coefficient controlling the relative importance of change\_loss in the training objective function. (Default to be 5).


```bash				    
fold_number = 50  # number of folds the hold-out cv runs
data_fraction = 0.8 # fraction of data used in each fold
cross_validated_clustering(train_data, ncluster, fold_number, data_fraction, start_saving_epoch, max_epoch,\
					    output_dir, WD, AQ, cluster_loss, consensus_tpype, covariate=covariate)
```

**cross\_validated\_clustering** performs clustering with hold-out cross validation. It is the ***main*** function for clustering. Since the CV process may take long training time on a normal desktop computer, the function enables early stop and later resumption. Users can set ***stop\_fold*** to be early stopping point and ***start\_fold*** depending on previous stopping point. The function automatically saves an csv file with clustering results. Two metrics are also provided for hyper-parameter selection: the mean ARI value (i.e., agreements of clusters among all folds), and the mean N\_Sig\_SNPs (significant SNPs in test sets).

```					    
model_dirs = ['PATH_TO_CHECKPOINT1','PATH_TO_CHECKPOINT2',...] #list of paths to previously saved checkpoints (with name 'converged_model_foldk' after cv process)
cluster_label, cluster_probabilities, _, _ = clustering_result(model_dirs, 'highest_matching_clustering', train_data, covariate)
```
**clustering\_result** is a function used for clustering patient data using previously saved models. **phenotype_data** and **covariate** (optional) should be panda dataframe with same format as introduced before. Only PT data (can be inside or outside of training set), for which the user want to derive cluster memberships, need to be provided with diagnoses set to be 1. **gene_data** is not required when applying the trained models. ***The function returns cluster labels of PT data following the order of PT in the provided dataframe.***


## Citation
If you use this package for research, please cite the following paper:


```bash
@article{yang2021BrainHeterogeneity,
author = {Yang, Zhijian and Nasrallah, Ilya M. and Shou, Haochang and Wen, Junhao and Doshi, Jimit and Habes, Mohamad and Erus, Guray and Abdulkadir, Ahmed and Resnick, Susan M. and Albert, Marilyn S. and Maruff, Paul and Fripp, Jurgen and Morris, John C. and Wolk, David A. and Davatzikos, Christos and {iSTAGING Consortium} and {Baltimore Longitudinal Study of Aging (BLSA)} and {Alzheimer’s Disease Neuroimaging Initiative (ADNI)}},
year = {2021},
month = {12},
pages = {},
title = {A deep learning framework identifies dimensional representations of Alzheimer’s Disease from brain structure},
volume = {12},
journal = {Nature Communications},
doi = {10.1038/s41467-021-26703-z}
}
```


