# plot1cell: a package for advanced single cell data visualization

This package allows users to visualize the single cell data on the R object or output files generated by Seurat. It is currently under active development. 

## Installation
plot1cell R package can be easily installed from Github using devtools. Please make sure you have installed <a href="https://satijalab.org/seurat/">Seurat 4.0</a>, <a href="https://github.com/jokergoo/circlize">circlize</a> and <a href="https://github.com/jokergoo/ComplexHeatmap">ComplexHeatmap</a> packages.

```
devtools::install_github("TheHumphreysLab/plot1cell")
## or the development version, devtools::install_github("HaojiaWu/plot1cell")
```

## Usage
We provide some example codes to help generate figures from user's provided Seurat object. The Seurat object input to plot1cell should be a final object with completed clustering and cell type annotation. If a seurat object is not available, we suggest to use the demo data from Satija's lab (https://satijalab.org/seurat/articles/integration_introduction.html). To demonstrate the plotting functions in plot1cell, we re-created a Seurat object from our recent paper <a href="https://www.pnas.org/doi/10.1073/pnas.2005477117">Kirita et al, PNAS 2020</a> by integrating the count matrices we uploaded to GEO (GSE139107).
```
library(plot1cell)
iri.integrated <- Install.example() 

# Please note that this Seurat object is just for demo purpose and 
# is not exactly the same as we published on PNAS.
# It takes about 2 hours to run in a linux server with 500GB RAM and 32 CPU cores.
# You can skip this step and use your own Seurat object instead
```

### 1. Circlize plot to visualize cell clustering and meta data
This circlize plot was inspired by the data visualization in a published paper (Figure1, https://www.nature.com/articles/s41586-021-03775-x) from Linnarsson's lab.

```
###Check and see the meta data info on your Seurat object
colnames(iri.integrated@meta.data)  

###Prepare data for ploting
circ_data <- prepare_circlize_data(iri.integrated, scale = 0.8 )
set.seed(1234)
cluster_colors<-rand_color(length(levels(iri.integrated)))
group_colors<-rand_color(length(names(table(iri.integrated$Group))))
rep_colors<-rand_color(length(names(table(iri.integrated$orig.ident))))

###plot and save figures
png(filename =  'circlize_plot.png', width = 6, height = 6,units = 'in', res = 300)
plot_circlize(circ_data,do.label = T, pt.size = 0.01, col.use = cluster_colors ,bg.color = 'white', kde2d.n = 200, repel = T, label.cex = 0.6)
add_track(circ_data, group = "Group", colors = group_colors, track_num = 2) ## can change it to one of the columns in the meta data of your seurat object
add_track(circ_data, group = "orig.ident",colors = rep_colors, track_num = 3) ## can change it to one of the columns in the meta data of your seurat object
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/circlize_plot.png) <br />

### 2. Dotplot to show gene expression across groups
Here is an example to use plot1cell to show one gene expression across different cell types across groups.
```
png(filename =  'dotplot_single.png', width = 4, height = 6,units = 'in', res = 100)
complex_dotplot_single(seu_obj = iri.integrated, feature = "Havcr1",groupby = "Group")
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/dotplot_single.png) <br />
If the group factor can be classified by another factor, complex_dotplot_single allows splitting the group factor by another group factor too. Here is an example for demo.
```
iri.integrated@meta.data$Phase<-plyr::mapvalues(iri.integrated@meta.data$Group, from = levels(iri.integrated@meta.data$Group), to = c("Healthy",rep("Injury",3), rep("Recovery",2)))
iri.integrated@meta.data$Phase<-as.character(iri.integrated@meta.data$Phase)
png(filename =  'dotplot_single_split.png', width = 4, height = 6,units = 'in', res = 100)
complex_dotplot_single(iri.integrated, feature = "Havcr1",groupby = "Group",splitby = "Phase")
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/dotplot_single_split.png) <br />

plot1cell also allows visualization of multiple genes in dotplot format. Here is an example.
```
png(filename =  'dotplot_multiple.png', width = 10, height = 4,units = 'in', res = 300)
complex_dotplot_multiple(seu_obj = iri.integrated, features = c("Slc34a1","Slc7a13","Havcr1","Krt20","Vcam1"),groupby = "Group", celltypes = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"))
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/dotplot_multiple.png) <br />

### 3. Violin plot to show gene expression across groups
#### One gene/one group factor violin plot:
```
png(filename =  'vlnplot_single.png', width = 4, height = 6,units = 'in', res = 100)
complex_vlnplot_single(iri.integrated, feature = "Havcr1", groups = "Group",celltypes   = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"))
dev.off()
```

![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/vlnplot_single.png) <br />

Similar to complex_dotplot_single, the complex_vlnplot_single function also allows splitting the group factor by another factor with the argument "splitby".
```
png(filename =  'vlnplot_single_split.png', width = 4, height = 6,units = 'in', res = 100)
complex_vlnplot_single(iri.integrated, feature = "Havcr1", groups = "Group",celltypes   = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"), splitby = "Phase")
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/vlnplot_single_split.png) <br />
#### One gene/multiple group factors violin plot:
```
png(filename =  'vlnplot_multiple.png', width = 6, height = 6,units = 'in', res = 100)
complex_vlnplot_single(iri.integrated, feature = "Havcr1", groups = c("Group","Replicates"),celltypes   = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"), font.size = 10)
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/vlnplot_multiple.png) <br />

Each group factor can be further splitted by its own factor by setting the splitby argument. Note that in this case, the order of the group factors needs to match the order of splitby factors. For example:
```
iri.integrated@meta.data$ReplicateID<-plyr::mapvalues(iri.integrated@meta.data$Replicates, from = names(table((iri.integrated@meta.data$Replicates))), to = c(rep("Rep1",3),rep("Rep2",3), rep("Rep3",1)))
iri.integrated@meta.data$ReplicateID<-as.character(iri.integrated@meta.data$ReplicateID)

png(filename =  'vlnplot_multiple_split.png', width = 7, height = 5,units = 'in', res = 200)
complex_vlnplot_single(iri.integrated, feature = "Havcr1", groups = c("Group","Replicates"),
                        celltypes   = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"), 
                        font.size = 10, splitby = c("Phase","ReplicateID"), pt.size=0.05)
dev.off()

### In this example, "Phase" is a splitby factor for "Group" and "ReplicateID" is a splitby factor for "Replicates".

```

![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/vlnplot_multiple_split.png) <br />

Note that the Replicates group here is just for showcase purpose. This is not a meaning group ID in our snRNA-seq dataset.

#### Multiple genes/one group factor violin plot:
```
png(filename =  'vlnplot_multiple_genes.png', width = 6, height = 6,units = 'in', res = 300)
complex_vlnplot_multiple(iri.integrated, features = c("Havcr1",  "Slc34a1", "Vcam1",   "Krt20"  , "Slc7a13", "Slc5a12"), celltypes = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"), group = "Group", add.dot=T, pt.size=0.01, alpha=0.01, font.size = 10)
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/vlnplot_multiple_genes.png) <br />

#### Multiple genes/multiple group factors.
The violin plot will look too messy in this scenario so it is not included in plot1cell. It is highly recommended to use the complex_dot_plot instead. <br />

### 4. Umap geneplot across groups
```
png(filename =  'data/geneplot_umap.png', width = 8, height = 6,units = 'in', res = 100)
complex_featureplot(iri.integrated, features = c("Havcr1",  "Slc34a1", "Vcam1",   "Krt20"  , "Slc7a13"), group = "Group", select = c("Control","12hours","6weeks"), order = F)
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/geneplot_umap.png) <br />

### 5. ComplexHeatmap to show unique genes across groups
plot1cell can directly identify the condition specific genes in a selected cell type and plot those genes using ComplexHeatmap. An example is shown below:
```
iri.integrated$Group2<-plyr::mapvalues(iri.integrated$Group, from = c("Control", "4hours",  "12hours", "2days",   "14days" , "6weeks" ),
to = c("Ctrl","Hr4","Hr12","Day2", "Day14","Wk6"))
iri.integrated$Group2<-factor(iri.integrated$Group2, levels = c("Ctrl","Hr4","Hr12","Day2", "Day14","Wk6"))
png(filename =  'heatmap_group.png', width = 4, height = 8,units = 'in', res = 100)
complex_heatmap_unique(seu_obj = iri.integrated, celltype = "NewPT2", group = "Group2",gene_highlight = c("Slc22a28","Vcam1","Krt20","Havcr1"))
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/heatmap_group.png) <br />

### 6. Upset plot to show the unique and shared DEGs across groups.

```
png(filename =  'upset_plot.png', width = 8, height = 4,units = 'in', res = 300)
complex_upset_plot(iri.integrated, celltype = "NewPT2", group = "Group", min_size = 10, logfc=0.5)
dev.off()
```
![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/upset_plot.png) <br />

### 7. Cell proportion change across groups

```
png(filename =  'cell_fraction.png', width = 8, height = 4,units = 'in', res = 300)
plot_cell_fraction(iri.integrated,  celltypes = c("PTS1" ,   "PTS2"  ,  "PTS3"  ,  "NewPT1" , "NewPT2"), groupby = "Group", show_replicate = T, rep_colname = "orig.ident")
dev.off()
```

![alt text](https://github.com/HaojiaWu/Plot1cell/blob/master/data/cell_fraction.png) <br />

### 8. Other ploting functions
There are other functions for plotting/data processing in plot1cell. 
```
help(package = plot1cell)
```
Many more functions will be added in the future package development. For questions, please raise an issue in this github page or contact <a href="https://humphreyslab.com">TheHumphreysLab</a>. 

### 9. Attributions
This package uses many methods from Seurat (https://github.com/satijalab/seurat) to process the data for ploting. The circlize and heatmap plots were generated by the circlize (https://github.com/jokergoo/circlize) and ComplexHeatmap (https://github.com/jokergoo/ComplexHeatmap) packages. The Upset plot was generated by the ComplexUpset package (https://github.com/krassowski/complex-upset). Most of the graphs were generated by ggplot2 (https://github.com/tidyverse/ggplot2). The package benefits from the following dependencies.

```
    Seurat,
    plotly,
    circlize,
    dplyr,
    ggplot2,
    MASS,
    scales,
    progress,
    RColorBrewer,
    grid,
    grDevices,
    biomaRt,
    reshape2,
    ggbeeswarm,
    purrr,
    ComplexUpset,
    matrixStats,
    DoubletFinder,
    methods,
    data.table,
    Matrix,
    hdf5r,
    loomR,
    GenomeInfoDb,
    EnsDb.Hsapiens.v86,
    cowplot,
    rlang,
    GEOquery,
    simplifyEnrichment,
    wordcloud,
    ComplexHeatmap
```
