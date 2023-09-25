# Convert between h5ad & Seurat object

**Important note:** You may have come across the old conversion vignettes and steps since they still come up first in Google searches and critical new info is buried:

These vignettes for older Seurat versions (2 and 3) **do not work** in Seurat 4, as they moved support for h5ad/anndata to SeuratDisk:
- [Conversion in Seurat2](https://satijalab.org/seurat/archive/v2.4/conversion_vignette)
- [Conversion in Seurat3](https://satijalab.org/seurat/archive/v3.1/conversion_vignette.html)

And this SeuratDisk vignette also is not reliable because [SeuratDisk is no longer being maintained](https://github.com/satijalab/seurat/issues/7730):
- [SeuratDisk vignette: convert h5ad to Seurat](https://mojaveazure.github.io/seurat-disk/articles/convert-anndata.html)

This is important because it appears the h5ad file format changed at some point in ways important to SeuratDisk, and SeuratDisk has not been updated to support the new file type. It's possible bugs may be introduced if you try to use the old methods to convert your files.


## Use `sceasy`

Here is the [GitHub site for sceasy](https://github.com/cellgeni/sceasy)... and it really is easy.

I installed through RStudio. The steps on the GitHub README are straightforward; see there for details. Here is what you need if you just plan to convert between AnnData/h5ad and Seurat (not Loom, etc.):

1. [R] - Install `sceasy`, `reticulate`, `LoomExperiment`, and `SingleCellExperiment`

2. [bash] - Create a new conda environment for `anndata`. You could just install packages in the base environment, but Python has so many issues with package dependency conflicts that it's best to have projects running in their own envs.

```
# Create env named anndata (or whatever you want)
$ conda create --name anndata

# Activate the new env so you can install packages
$ conda activate anndata
```

3. [bash] - Install `anndata` in your new conda env. (If using Loom files, also install `loompy`)

```
(anndata) $ conda install anndata -c bioconda
```

Now you're ready to convert your files in R using `sceasy`. Usage is also straightforward and is detailed on the GitHub README.

Be sure to tell R to use your new conda environment when running `sceasy` - that's what this part of the R code does (if your env is named anndata):

```
use_condaenv("anndata")
```
