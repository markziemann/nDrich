# Mitch
Mitch is a tool for multi-dimensional enrichment analysis. At it's heart, it uses a rank-MANOVA based statistical approach to detect sets of genes that exhibit enrichment in the multidimensional space as compared to the background. Mitch is useful for pathway analysis of profiling studies with two to or more contrasts, or in studies with multiple omics profiling, for example proteomic, transcriptomic, epigenomic analysis of the same samples.

## Installation
```
install.packages("devtools")
library("devtools")
devtools::install_github("markziemann/Mitch")
library("mitch")
```

## Workflow overview
### Importing gene sets
Mitch has a function to import GMT files to R lists, which was adapted from work by [Yu et al, 2012](https://dx.doi.org/10.1089%2Fomi.2011.0118) in the [clusterProfiler](http://bioconductor.org/packages/release/bioc/html/clusterProfiler.html) package. For example:
```
genesets<-gmt_import("Reactome.gmt")
```
### Importing profiling data
Mitch accepts pre-ranked data supplied by the user, but also has a function called `mitch_import` for importing tables generated by Limma, edgeR, DESeq2, ABSSeq and Sleuth. By default, only the genes that are detected in all contrasts are included, but this behaviour can be modified. The below example imports two edgeR tables called "dge1" and "dge2".
```
x<-list("dge1"=dge1,"dge2"=dge2)
y<-mitch_import(x,"edger")
```
By default, differential gene activiy is scored using the directional nominal p-value.

S=-log10(p-value) * sgn(logFC)

If this is not desired, then users can perform their own custom scoring procedure.

`mitch_import` also accepts a two-column table that relates gene identifiers in the profiling data to those in the gene sets. `?mitch_import` provides more instructions on using this feature.
### Calculating enrichment
The `mitch_calc` function performs multivariate enrichment analysis of the supplied gene sets in the scored profiling data.  At its simpest form `mitch_calc` function accepts the scored data as the first argument and the genesets as the second argument. Users can prioritise enrichments based on small adjusted p-values, or by the observed effect size (magnitude of the enrichment score).
```
res<-mitch_calc(y,genesets,priority="significance")
res<-mitch_calc(y,genesets,priority="effect")
```
By default, `mitch_calc` uses mclapply to speed up calculations on all but one available CPU threads. This behaviour can be modified by setting the `cores` to a desred number.
```
res<-mitch_calc(y,genesets,priority="significance",cores=4)
```
By default, gene sets with fewer than 10 members present in the profiling data are discarded. This threshold can be modified using the `minsetsize` option. There is no upper limit of gene set size.
```
res<-mitch_calc(y,genesets,priority="significance",minsetsize=20)
```

Optionally, users can specify bootstrapping to estimate the lower 95% confidence interval of the s.distance (effect size). This is done by randomly resampling the gene set with replacement. This adds significantly to the execution time, so it is not advised for "first-pass" analysis. A reasonable bootstrap number is 1000.
```
res<-mitch_calc(y,genesets,priority="significance",bootstraps=1000)
```
If bootstraps are specified, then the results can be prioritised by the lower confidence interval of the s.distance:
```
res<-mitch_calc(y,genesets,priority="confidence",bootstraps=1000)
```

By default, in downstream visualisation steps, charts are made from the top 50 gene sets, but this can be modified using the `resrows` option. 
```
res<-mitch_calc(y,genesets,priority="significance",resrows=100)
```
