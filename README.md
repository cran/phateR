phateR
================

<!-- README.md is generated from README.Rmd. Please edit that file -->

[![Latest PyPI
version](https://img.shields.io/pypi/v/phate.svg)](https://pypi.org/project/phate/)
[![Latest CRAN
version](https://img.shields.io/cran/v/phateR.svg)](https://cran.r-project.org/package=phateR)
[![Travis CI
Build](https://api.travis-ci.com/KrishnaswamyLab/phateR.svg?branch=master)](https://travis-ci.com/KrishnaswamyLab/phateR)
[![Read the
Docs](https://img.shields.io/readthedocs/phate.svg)](https://phate.readthedocs.io/)
[![Nature Biotechnology Publication](https://zenodo.org/badge/DOI/10.1038/s41587-019-0336-3.svg)](https://www.nature.com/articles/s41587-019-0336-3)
[![Twitter](https://img.shields.io/twitter/follow/KrishnaswamyLab.svg?style=social&label=Follow)](https://twitter.com/KrishnaswamyLab)
[![Github
Stars](https://img.shields.io/github/stars/KrishnaswamyLab/PHATE.svg?style=social&label=Stars)](https://github.com/KrishnaswamyLab/PHATE/)

This R package provides an implementation of the [PHATE dimensionality
reduction and visualization
method](https://www.nature.com/articles/s41587-019-0336-3).

For a thorough overview of the PHATE visualization method, please see
the [Nature Biotechnology publication](https://www.nature.com/articles/s41587-019-0336-3).

For our Python and Matlab implementations, please see
[KrishnaswamyLab/PHATE](https://github.com/KrishnaswamyLab/PHATE).

## Table of Contents

  - [Installation](#installation)
      - [Installation from CRAN and
        PyPi](#installation-from-cran-and-pypi)
      - [Installation with devtools and
        <code>reticulate</code>](#installation-with-devtools-and-reticulate)
      - [Installation from source](#installation-from-source)
  - [Quick Start](#quick-start)
  - [Tutorial](#tutorial)
  - [Issues](#issues)
      - [FAQ](#faq)
      - [Help](#help)

## Installation

In order to use PHATE in R, you must also install the Python package.

If `python` or `pip` are not installed, you will need to install them.
We recommend [Miniconda3](https://conda.io/miniconda.html) to install
Python and `pip` together, or otherwise you can install `pip` from
<https://pip.pypa.io/en/stable/installing/>.

#### Installation from CRAN and PyPi

First install `phate` in Python by running the following code from a
terminal:

``` bash
pip install --user phate
```

Then install `phateR` from CRAN by running the following code in R:

``` r
install.packages("phateR")
```

#### Installation with `devtools` and `reticulate`

The development version of PHATE can be installed directly from R with
`devtools`:

``` r
if (!suppressWarnings(require(devtools))) install.packages("devtools")
reticulate::py_install("phate", pip=TRUE)
devtools::install_github("KrishnaswamyLab/phateR")
```

#### Installation from source

The latest source version of PHATE can be accessed by running the
following in a terminal:

``` bash
git clone --recursive git://github.com/KrishnaswamyLab/PHATE.git
cd PHATE/Python
python setup.py install --user
cd ../phateR
R CMD INSTALL
```

If the `phateR` folder is empty, you have may forgotten to use the
`--recursive` option for `git clone`. You can rectify this by running
the following in a terminal:

``` bash
cd PHATE
git submodule init
git submodule update
cd Python
python setup.py install --user
cd ../phateR
R CMD INSTALL
```

## Quick Start

If you have loaded a data matrix `data` in R (cells on rows, genes on
columns) you can run PHATE as follows:

``` r
library(phateR)
data_phate <- phate(data)
```

phateR accepts R matrices, `Matrix` sparse matrices, `data.frame`s, and
any other data type that can be converted to a matrix with the function
`as.matrix`.

## Tutorial

This is a basic example running `phate` on a highly branched example
dataset that is included with the package. You can read a tutorial on
running PHATE on single-cell RNA-seq at
<http://htmlpreview.github.io/?https://github.com/KrishnaswamyLab/phateR/blob/master/inst/examples/bonemarrow_tutorial.html>
or in `inst/examples`. Running this tutorial from start to finish should
take approximately 3 minutes.

First, let’s load the tree data and examine it with PCA.

``` r
library(phateR)
#> Loading required package: Matrix
data(tree.data)
plot(prcomp(tree.data$data)$x, col=tree.data$branches)
```

<img src="man/figures/README-example-data-1.png" width="100%" />

Now we run PHATE on the data. We’ll just go ahead and try with the
default parameters.

``` r
# runs phate
tree.phate <- phate(tree.data$data)
summary(tree.phate)
#> PHATE embedding
#> k = 5, alpha = 40, t = auto
#> Data: (3000, 100)
#> Embedding: (3000, 2)
```

Let’s plot the results.

``` r
# plot embedding
palette(rainbow(10))
plot(tree.phate, col = tree.data$branches)
```

<img src="man/figures/README-plot-results-1.png" width="100%" />

Good news\! Our branches separate nicely. However, most of the
interesting activity seems to be concentrated into one region of the
plot - in this case we should try the square root potential instead by
using `gamma=0`. We can also try increasing `t` to make the structure a
little clearer - in this case, because synthetic data in unusually
structured, we can use a very large value, like 120, but in biological
data the automatic `t` selection is generally very close to ideal. Note
here that if we pass our previous result in with the argument `init`, we
won’t have to recompute the diffusion operator.

``` r
# runs phate with different parameters
tree.phate <- phate(tree.data$data, gamma=0, t=120, init=tree.phate)
# plot embedding
palette(rainbow(10))
plot(tree.phate, col = tree.data$branches)
```

<img src="man/figures/README-adjust-parameters-1.png" width="100%" />

We can also pass the PHATE object directly to `ggplot`, if it is
installed.

``` r
library(ggplot2)
#> Warning: package 'ggplot2' was built under R version 3.5.3
ggplot(tree.phate, aes(x=PHATE1, y=PHATE2, color=tree.data$branches)) +
  geom_point()
```

<img src="man/figures/README-ggplot-1.png" width="100%" />

## Issues

### FAQ

  - **Should genes (features) by rows or columns?**

To be consistent with common dimensionality reductions such as PCA
(`stats::prcomp`) and t-SNE (`Rtsne::Rtsne`), we require that cells
(observations) be rows and genes (features) be columns of your input
data.

  - **I have installed PHATE in Python, but phateR says it is not
    installed\!**

Check your `reticulate::py_discover_config("phate")` and compare it to
the version of Python in which you installed PHATE (run `which python`
and `which pip` in a terminal.) Chances are `reticulate` can’t find the
right version of Python; you can fix this by adding the following line
to your `~/.Renviron`:

`PATH=/path/to/my/python`

You can read more about `Renviron` at
<https://CRAN.R-project.org/package=startup/vignettes/startup-intro.html>.

### Help

Please let us know of any issues at the [GitHub
repository](https://github.com/KrishnaswamyLab/phateR/issues). If you
have any questions or require assistance using PHATE, please read the
documentation at <https://CRAN.R-project.org/package=phateR/phateR.pdf>
or by running `help(phateR::phate)` or contact us at
<https://krishnaswamylab.org/get-help>.
