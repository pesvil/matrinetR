![My Image](network.jpeg)
MatriNet is a recently published database and analysis platform for the exploration of extracellular matrix (ECM) protein networks across tumors with interactive tools embedded into graphical user interfaces (available at www.matrinet.org).  This package is an open-source repository and a modular extension to MatriNet that enables local operations on the same data sources with enhanced customizations, previews features before their online implementation, and promotes collaborative efforts and community-driven solutions for the MatriNet ecosystem.

If you have any questions or suggestions, please feel free to contact us.

<!-- GETTING STARTED -->
## Install the package
The developmental version of MatriNet R package is freely available and can be installed as follows:
```r
require(devtools)
devtools::install_github("KontioJuho/matrinetR")
``` 


<!-- USAGE EXAMPLES -->
## Matrinet objects
The MatrinetR library has two major objects: matridata and matrigraph. The matridata is a list consisting an object *group* for each sample group (e.g. by tumor). Then each group object, is consisting of three different preprocessed gene/protein dataframes: 

Continuous:  log2-transformed gene/protein expression data

Discrete: Discretized gene/protein expression data (up/down regulation)

Profile: Frequency distributions of discretized gene/protein expression levels.

These three different types of dataframes are used in different ways in the network estimation process. All of the preceding network data, e.g. known interactions and  gene-specific annotations are combined into a single matrigraph entity that will be updated based on network estimates' outcomes.

```r
library(matrinetR)

?matrixDB

```

## MatrinetR for gene expression data 

MatrinetR contains pre-processed gene expression and clinical data for 23 tumor types from The Cancer Genome Atlas (TCGA) and corresponding normal (GTEx) tissues from the Genotype-Tissue Expression (GTEx) database, as well as immunohistochemistry staining profiles from The Human Protein Atlas (THPA).

```r
?matrisome_TCGA
?matrisome_GTEx
```


## Tutorial for new features

The MatrinetR workflow begins by defining the target genes and ensuring that they are accessible in the gene/protein data set. In this example, our target genes are all genes in the matrixDB interaction dataset and it seems that 363 of them are available in both GTEx and TCGA cohorts.

```r

matrisome_genes <- unique(c(matrixDB$gene.x, matrixDB$gene.y))
cancers <- c("brca", "ov", "prad")


valid_genesTCGA <- available_genes(target_genes = matrisome_genes,
                                   data = matrisome_TCGA[cancers])
                                   
valid_genesGTEx <- available_genes(target_genes = matrisome_genes, 
                                   data = matrisome_GTEx[cancers])


valid_genes <- intersect(valid_genesTCGA$available_zero_NAs,
                         valid_genesGTEx$available_zero_NAs)



```
The next step is to the specify matridata objects for each cohort which automatically extacts available variables and checks if they are valid for the network estimation process. A variable is selected if it is measured on 95% of all samples in each selected cancer type and cohort (i.e. almost complete cases only).  If the number of missing values is less than 5% of all observations, the default imputation method is to use a variable and group specific median. 

```r

matridata_GTEx <- matrinet_data(data = matrisome_GTEx[cancers],
                                quantiles = discretization_levels,
                                genenames = valid_genes)


matridata_TCGA <- matrinet_data(data = matrisome_TCGA[cancers],
                                quantiles = discretization_levels,
                                genenames = valid_genes)

```


After the data is prepared, the next step is  to combine all of the preceding data annotations into a single matrigraph object that will be subsequently updated based on the network estimations' outcomes.

```r

matrigraph_TCGA <- matrinet_graph(matridata = matridata_TCGA,
                                  prior_topology = matrixDB_adjacency[valid_genes,
                                                                      valid_genes])

matrigraph_GTEx <- matrinet_graph(matridata = matridata_GTEx ,
                                  prior_topology = matrixDB_adjacency[valid_genes,
                                                                      valid_genes])



```
Matridata and matrigraph objects are the main two inputs to the actual network estimation step, handled by the function matrinet_estimate.  The function returns an updated matrigraph object that includes all of the edge weights and other relevant information about the estimation process.

```r

matrinet_TCGA <- matrinet_estimate(matrigraph = matrigraph_TCGA,
                                   matridata = matridata_TCGA)

matrinet_GTEx <- matrinet_estimate(matrigraph = matrigraph_GTEx,
                                   matridata = matridata_GTEx)

```
_For more examples, please refer to the [Documentation]()_

<p align="right">(<a href="#top">back to top</a>)</p>
