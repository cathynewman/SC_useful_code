# Convert between h5ad & Seurat object

**Important note:** You may have come across the old conversion vignettes and steps since they still come up first in Google searches and critical new info is buried:

These vignettes for older Seurat versions (2 and 3) **do not work** in Seurat 4, as they moved support for h5ad/anndata to SeuratDisk:
- Conversion in [Seurat2](https://satijalab.org/seurat/archive/v2.4/conversion_vignette) or [Seurat3](https://satijalab.org/seurat/archive/v3.1/conversion_vignette.html)

And [this SeuratDisk vignette](https://mojaveazure.github.io/seurat-disk/articles/convert-anndata.html) also is not reliable because [SeuratDisk is no longer being maintained](https://github.com/satijalab/seurat/issues/7730).

This is important because it appears the h5ad file format changed at some point in ways important to SeuratDisk, and SeuratDisk has not been updated to support the new file type. It's possible bugs may be introduced if you try to use the old methods to convert your files.


## Use `sceasy`

Here is the [GitHub site for sceasy](https://github.com/cellgeni/sceasy)... ~~and it really is easy~~ (*no it isn't*).

**UPDATE JULY 2024:** Something broke `sceasy` for me, not sure what. Here is what is currently working for me:
* R: Seurat v.4.4.0
* R: sceasy v.0.0.7
* R: reticulate v.1.37.0 (older version throws an import error, so be sure to update)
* Python: create conda env with python v.3.11 (not sure if version is critical)
* anndata v.0.9.1 (this is important; default version installed is older)
* numpy v. 1.26.4 (important to install version < 2.0.0)

I installed the R packages in RStudio. The steps on the GitHub README are straightforward; see there for details. Here is what you need if you just plan to convert between AnnData/h5ad and Seurat (not Loom, etc.):

1. [R] - Install `sceasy`, `reticulate`, `LoomExperiment`, and `SingleCellExperiment`

2. [bash] - Create a new conda environment for `anndata`. You could just install packages in the base environment, but Python has so many issues with package dependency conflicts that it's best to have projects running in their own envs.

```
# Create env named anndata (or whatever you want)
# Specify python version
$ conda create --name anndata python=3.11

# Activate the new env so you can install packages
$ conda activate anndata
```

3. [bash] - Install `anndata` in your new conda env. (If using Loom files, also install `loompy`.)

```
(anndata) $ conda install anndata=0.9.1
```

4. Downgrade `numpy` from 2.x using the `pip3` found in your conda environment specifically. Change the path below to your path to `anaconda3/envs/yourenv/bin/pip3`. If you do not specify and instead use `pip install`, default pip will not correctly install into the conda environment.

```
/Users/newman/anaconda3/envs/anndata/bin/pip3 install "numpy<2.0.0"
```

Now you're ready to convert your files in R using `sceasy`. Usage is also straightforward and is detailed on the GitHub README.

Be sure to tell R to use your new conda environment when running `sceasy` - that's what this part of the R code does (if your env is named anndata):

```
use_condaenv("anndata")
```

### Options for converting Seurat to AnnData

There are a few additional conversion options not included in the sceasy README, found in the R code. In particular, options for which assay and which slot is included. It looks like each exported AnnData object includes one assay, a main slot, and other slots as "layers" - so for example, you might export the RNA assay data slot (main) with the counts slot as a layer, OR export the integrated assay data slot (counts matrix is empty). **Default is RNA assay, data slot, no layers.**

Seurat to AnnData - **RNA assay, data slot only**. This is the default behavior if `assay`, `main_layer`, and `transfer_layers` arguments are not included.

```
sceasy::convertFormat(allRats, outFile = "allRats_souped_noDoub_data.h5ad",
                      from = "seurat", to = "anndata")
```

Seurat to AnnData - **RNA assay, data + counts**

Note: The counts layer doesn't seem to be very accessible in `scanpy` in the way that it is as an alternate slot in Seurat. Probably easiest to instead export individual AnnData objects with `main_layer` either **data** or **counts** and omit the `transfer_layers` argument.

```
sceasy::convertFormat(allRats, outFile = "allRats_souped_noDoub_RNA.h5ad",
                      assay = "RNA", main_layer = "data",
                      transfer_layers = "counts",
                      from = "seurat", to = "anndata")
```

Seurat to AnnData - **integrated assay (data slot)**

```
sceasy::convertFormat(allRats, outFile = "allRats_souped_noDoub_integrated.h5ad",
                      assay = "integrated", main_layer = "data",
                      from = "seurat", to = "anndata")
```
