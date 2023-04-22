![alt text](img/spectra_img.png?raw=true)


## Overview

SPECTRA takes in a single cell gene expression matrix, cell type annotations, and a set of pathway annotations to fit the data to. 

```
gene_set_annotations = {
	@@ -28,29 +25,40 @@ gene_set_annotations = {


# Installation 

A pypi package will be available soon. For installation, you can add spectra to pip:  

```
pip install git+https://github.com/dpeerlab/spectra
```


# Tutorial

We provide a full tutorial how to run the basic Spectra model here: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/dpeerlab/spectra/blob/main/notebooks/Spectra_Colaboratory_tutorial.ipynb)

You can use log1p transformed median library size normalized data. For leukocyte data, we recommend using [scran normalization](https://doi.org/10.1186/s13059-016-0947-7). We provide a [tutorial](https://github.com/dpeerlab/spectra/blob/main/notebooks/scran_preprocessing.ipynb).


# scRNAseq Knowledge Base

Check out our scRNAseq knowledge base [Cytopus :octopus:](https://github.com/wallet-maker/cytopus) to retrieve *Spectra* input gene sets adapted to the cell type composition in your data.


# Short Tutorial

We start by importing spectra. The easiest way to run spectra is to use the `est_spectra` function in the `spectra` module, as shown below. The default behavior is to set the number of factors equal to the number of gene sets plus one. However, this can be modified by passing an integer e.g. `L = 20` as an argument to the function or a dictionary that maps cell type to an integer per cell type. We provide a method for estimating the number of factors directly from the data by bulk eigenvalue matching analysis, which is detailed further below. 

```
from spectra import spectra as spc
model = spc.est_spectra(adata = adata,  gene_set_dictionary = gene_set_dictionary, cell_type_key = "cell_type", use_highly_variable = True, lam = 0.01)
```


## Accessing model parameters
To access finer grained information about the model fit, we can look at the attributes of the model object directly. Model parameters can be accessed with functions associated with the model object

```
model.return_eta_diag()
model.return_cell_scores()
model.return_factors() 
model.return_eta()
model.return_rho()
model.return_kappa()
model.return_gene_scalings()
```

Apart from cell scores and factors, we can also retrive a number of other parameters this way that are not by default added to the AnnData. Eta diag is the diagonal of the fitted factor-factor interaction matrix; however, its interpretation is that it measures the extent to which each factor is influenced by the prior information. In practice many of these values are zero, indicating that they are estimated without bias introduced by the annotation set. Eta is the full set of factor-factor interaction matrices, whose off diagonals measure the extent to which factors share the same genes. Rho and kappa are parameters that control the background rate of non-edges and edges respectively. These can be fixed throughout training (default) or estimated from the data by providing `rho = None` or `kappa = None` to the `est_spectra()` function  or to `model.train()`. Finally gene scalings are correction factors that normalize each gene based on its mean expression value. 

## Estimating the number of factors
To estimate the number of factors first, run
```
from spectra import K_est as kst
L = kst.estimate_L(adata, attribute = "cell_type", highly_variable = True)
```
## Fitting via EM 
For smaller problems we can use a memory intensive EM algorithm instead
```
X = adata.X 
A = binary adjacency matrix 
model = spc.SPECTRA_EM(X = X, A= A, T = 4)
model.fit()