# Convert h5ad to Seurat object

The steps needed depend on which version of **anndata** wrote the h5ad file, as the file format has changed over time (I don't know specifics).

Unfortunately, I have not found a way to see in a given file which version of anndata created it. So if you have an h5ad file that you don't know the origin of, you will have to trial-and-error your way through the steps below until you hit on something that works with your file.

## Helpful resources
I cobbled together most of the code below from these resources:
- [SeuratDisk vignette: convert h5ad to Seurat](https://mojaveazure.github.io/seurat-disk/articles/convert-anndata.html)
- [SeuratDisk open issue](https://github.com/mojaveazure/seurat-disk/issues/109)

Note that these vignettes for older Seurat versions (2 and 3) **do not work** in Seurat 4, as they moved support for h5ad/anndata to SeuratDisk (which may or may not still be maintained):
- [Conversion in Seurat2](https://satijalab.org/seurat/archive/v2.4/conversion_vignette)
- [Conversion in Seurat3](https://satijalab.org/seurat/archive/v3.1/conversion_vignette.html)

## AnnData
The current version of [anndata](https://anndata.readthedocs.io/en/latest/) as of this writing (Sept. 2023) is v.0.9.2.

It appears that the h5ad file format changed at some point in ways important to SeuratDisk and that SeuratDisk has not been updated to support the new file type.

I have not had luck importing any h5ad file with SeuratDisk if the file was written by an anndata version newer than **0.7.5**. We'll return to this later.

## 1. AnnData (h5ad) to h5Seurat intermediate [R]
The first step is to convert the AnnData (h5ad) file to h5Seurat using **SeuratDisk**. This creates an h5Seurat file in the working directory with the same filename as the h5ad file but extension `*.h5seurat`.

This part *should* work regardless of origin of the h5ad file.

```
library(Seurat)
library(SeuratDisk)

Convert("pbmc3k_0.7.5.h5ad", dest = "h5seurat")
```

## 2. h5Seurat to Seurat object [R]
This step imports the h5Seurat file created in step 1 and creates a Seurat object.

These two ways both work on h5ad files written by anndata 0.7.5 and might work with older anndata files. They **do not** work on newer anndata files (0.9.2).

Try first:

```
pbmc <- LoadH5Seurat("pbmc3k_0.7.5.h5seurat", assays = "RNA")
```

If that doesn't work, try this one:

```
pbmc <- LoadH5Seurat("pbmc3k_0.7.5.h5seurat", misc = FALSE)
```

If both of those fail, you may have an anndata file that's too new for SeuratDisk. Proceed to step 3...

## 3. Write new h5ad file with old AnnData [Python]
This is the only workaround I've found for newer anndata files that doesn't require exporting and importing the counts data and metadata separately. If you already have Python running via Anaconda or Miniconda, this is pretty straightforward.

**Bash code:**

1. Create a new conda environment

```
$ conda create --name anndata
$ conda activate anndata
```

2. Install pip

```
$ conda install pip
```

3. Install scanpy. Install required versions of numpy (1.24) and anndata 0.7.5.

```
$ pip install scanpy
$ conda install numpy=1.24 #required by scanpy
$ pip install --force-reinstall -v "anndata==0.7.5"
```

Now start **Python**

In Python, we will import the old h5ad file and write a new h5ad file. That's it.

```
>>> import scanpy as sc

# Read in old file
>>> adata = sc.read("pbmc3k_final.h5ad")

# Write new file
# Since we installed anndata 0.7.5, scanpy will use that version to write
>>> adata.write("pbmc3k_0.7.5.h5ad")
```

Now go back up to Step 1 and try with your new h5ad file.
