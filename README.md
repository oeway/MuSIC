# Multi-Scale Integrated Cell (MuSIC)
## What is MuSIC?
The eukaryotic cell is a multi-scale structure with modular organization across at least four orders of magnitude. Two central approaches for mapping this structure – protein fluorescent imaging and protein biophysical association – each generate extensive datasets, but of distinct qualities and resolutions that are typically treated separately. Here, we integrate immunofluorescence images in the Human Protein Atlas with affinity purification experiments from the BioPlex resource to create a unified hierarchical map of eukaryotic cell architecture. Integration involves configuring each approach to produce a general measure of protein distance, then calibrating the two measures using machine learning. The evolving map is called the Multi-Scale Integrated Cell (MuSIC).

<img src="https://github.com/idekerlab/MuSIC/blob/master/Figures/GitHub overview.png" width="400">

#### Portal page
https://nrnb.org/music/

#### Citing MuSIC
Please cite **[Qin et al., “Mapping cell structure across scales by fusing protein images and interactions”](https://www.biorxiv.org/cgi/content/short/2020.06.21.163709v1)**.


## Dependency:
Anaconda users please install relevant packages with the following command lines:
```
conda env create -f environment.yml
source activate music
```
We recommend using high performance computing to run the MuSIC pipeline on new datasets.


## Data embeddings
In MuSIC v1 study, we demonstrated the MuSIC pipeline with embeddings from immunofluorescence images and protein physical association data. However, the application of MuSIC pipeline is not limited to only these two data modalities. The key is to describe each protein with respect to the specific measurement platform. Of note, we recommend using the same number of dimensions for the embedding of each data modality.

#### HPA immunofluorescence image embedding
We are here providing the 1024-dimension embeddings for the 1,451 images used in MuSIC v1. Please refer to https://github.com/CellProfiling/densenet for image embedding code.

#### BioPlex Protein Embedding

```
python preprocess_node2vec.py --outprefix /path/to/output/folder/filePrefix 
```

## Compute protein similarities

### Analyze densenet results

This script splits images into six folds and determines similarities for each protein pair in each fold.

### Usage
```
python analyze_densenet.py --outprefix /path/to/output/folder/ --img_info /path/to/imageEmbeddingInfo.pkl --img_emd /path/to/imageEmbedding.pkl --prefix densenet_raw_1024D
```

### Required arguments for analyze_densenet.py:

`--outprefix` Full path to the folder where results will be saved in with unique file identifier.

`--img_info` Full path to pickled file containing info about each image (Example file all_hekfeature_dropMultiGeneEntry_dropNullAnnot.info.pkl included in Examples folder)

`--img_emd` Full path to pickled file containing image embeddings from densenet

`--prefix` Prefix to use for generated similarity files (E.g. densenet_raw_1024D)

### Anayze node2vec results

This script determines similarities for each protein pair from the node2vec embedding

### Usage
```
python analyze_node2vec.py --outprefix /path/to/output/folder/ --idx_to_name /path/to/node_idx_to_name.dict.pkl --emd /path/to/node2vec/emd.pkl --prefix dim_1024_p_2_q_1 --gene_list /path/to/genelist.npy
```

### Required arguments for analyze_node2vec.py:

`--outprefix` Full path to the folder where results will be saved in with unique file identifier.

`--idx_to_name` Full path to node_idx_to_name.dict.pkl file generated by preprocess_node2vec.py

`--emd` Full path to node2vec output emd file

`--prefix` Prefix to use for generated similarity files files (E.g. dim_1024_p_2_q_1)

`--gene_list` Full path to file containing genes to analyze (Example file img_x_ppi.661.genelist.npy is included in Examples folder)

## Format inputs for random forest

This script formats all data for the random forest training and testing algorithm. 

### Usage
```
python format_random_forest.py --outprefix /path/to/output/folder/ --go_input /path/to/go_input/human_go_cc.termStats --gene_list /path/to/genelist.npy  --n2v_prefix dim_1024_p_2_q_1 --densenet_prefix desenet_raw_1024D --n2v_gname /path/to/node2vec/dim_1024_p_2_q_1.gname.pkl  --densenet_emdfile /path/to/imageEmbedding.pkl --rest_gp_input /path/to/restgenepairs.npy
```

### Required arguments for format_random_forest.py

`--outprefix` Full path to the folder where results will be saved in with unique file identifier. 

`--go_input` Full path to file containing list of Gene Ontology terms size and genes (Example file human_go_cc.no_hpa.symbol.termStats in Examples folder)

`--gene_list` Full path to file containing genes to analyze (Example file img_x_ppi.661.genelist.npy included in Examples folder)

`--n2v_prefix` Prefix for node2vec generated similarity files (E.g. dim_1024_p_2_q_1)

`--densenet_prefix` Prefix for densenet generated similarity files (E.g. densenet_raw_1024D)

`--n2v_gname` Full path to output node2vec.gname.pkl file generated by analyze_node2vec.py

`--densenet_emdfile` Full path to pickled file containing image embeddings from densenet

`--rest_gp_input` Full path to file with gene pairs not in Gene Ontology (Example file rest_genepair.npy in Examples folder) 

## Train and test random forest

This script trains and tests a random forest model. Note that this step requires a large amount of memory - >100GB were used in this work. 

### Usage 
```
python train_RF.py --outputdir /path/to/output/folder/ --sdir /path/to/train/test/data --n_estimators 1000 --max_depth 30 --max_features auto --n2v_prefix dim_1024_p_2_q_1 --densenet_prefix desenet_raw_1024D
```
### Required arguments for train_RF.py
	
`--outprefix` Full path to the folder where results will be saved in with unique file identifier. 

`--sdir` Full path to the folder training and testing data was saved in format_random_forest.py 

`--n2v_prefix` Prefix for node2vec generated similarity files (E.g. dim_1024_p_2_q_1)

`--densenet_prefix` Prefix for densenet generated similarity files (E.g. densenet_raw_1024D)

`--n_estimators` Random forest number of estimators

`--max_depth` Random forest maximum depth

`--max_features` Random forest max features

## Random forest prediction

This script uses the random forest model to predict protein similarities and aggregates the results into an average predicted similarity.

### Usage 
```
python pred_RF.py --modelfname_prefix /path/to/folder/with/saved/models/densenet_raw_1024D.dim_1024_p_2_q_1.RF_maxDep_30_nEst_1000_maxFeat_auto --sdir /path/to/output/folder/train_test_data --outputdir /path/to/output/folder/ --rest_gp_input /path/to/restgenepairs.npy
```
### Required arguments for pred_RF.py

`--modelfname_prefix` Full path and prefix to model name prior to batch and fold number (E.g. /path/to/folder/with/saved/models/densenet_raw_1024D.dim_1024_p_2_q_1.RF_maxDep_30_nEst_1000_maxFeat_auto)
	
`--outprefix` Full path to the folder where results will be saved in with unique file identifier.

`--sdir` Full path to the folder training and testing data was saved in format_random_forest.py 

`--rest_gp_input` Full path to file with gene pairs not in Gene Ontology (Example file rest_genepair.npy in Examples folder) 

## Community detection and processing

### Specific Requirements
1. [CliXO v1.0](https://github.com/fanzheng10/CliXO-1.0)
2. [louvain-igraph v0.6.1](https://github.com/vtraag/louvain-igraph)
3. [DDOT](https://github.com/michaelkyu/ddot)
4. [alignOntology](https://github.com/mhk7/alignOntology)
5. networkx ≥v2.3
6. igraph

### Usage
```
python community_detection.py --outprefix /path/to/output/folder/filePrefix 
                              --path_to_clixo /path/to/CliXO/folder
                              --clixo_i /path/to/clixo/inputFile
                              --path_to_alignOntology /path/to/alignOntology/folder
                              
bash <outprefix>.sh
```
Hierarchy is written in file `<outprefix>.louvain.ddot` with specific protein assignment for each system available in file `<outprefix>.louvain.termStats`.

*Note: maximum clique finding is NP-hard, although utilized a heuristic approach, CliXO can still take a long time to finish.*

### Required arguments for community_detection.py:
`--outprefix` Full path to the folder where results will be saved in with unique file identifier.

`--path_to_clixo` Full path to CliXO folder.

`--clixo_i` Path to input similarity network for CliXO: a TSV file with three columns. The first two columns should be two strings for the node names (using numbers may cause problem); and the third column should be a value for the edge weight.

`--path_to_alignOntology` Full path to alignOntology folder.

### Optional arguments for community_detection.py:

`--clixo_a` CliXO -a flag: for the step size of hierarchy construction; usually, a smaller value will create "deeper" hierarchies with more levels from leaves to the root. (default: 0.1)

`--clixo_b` CliXO -b flag: for merging overlapping communities. Two existing communities will be merged if their similarity is above the threshold defined by this value. Usually a higher value will create a hierarchy with more smaller communities", which looks "broader". (default: 0.5)

`--clixo_m` CliXO -m flag: modularity cutoff. Communities lower than this threshold will be removed from the output. Increasing the value will reduce the number of communities. (default: 0)

`--clixo_z` CliXO -z flag: modularity cutoff. Communities lower than this threshold will be removed from the output. Increasing the value will reduce the number of communities. (default: 0)

`--clixo_s` CliXO -s flag: a cutoff of similarity score, if set, the program will terminate when it reaches this point, and stop looking for more terms from scores lower than this threshold. (default: 0)

`--minSystemSize` Minimum number of proteins requiring each system to have. (default: 4)

`--ci_thre` Threshold for containment index. Additional hierarchical parent-child containment relations will be assigned between pairs of systems having a containment index above this threshold. (default: 0.75)

`--ji_thre` Threshold for Jaccard index. System having Jaccard index above this threshold with its parent system will be removed. (default: 0.9)

`--niter` Number of iterations Louvain clustering will run to select partition with the best modularity. (default: 1000)

`--min_diff` Minimum difference in number of proteins required for every parent-child pair. (default: 1)

`--keep_all_files` When this flag is provided, all intermediate output files will be kept.

## Calibrate to physical distance

This script calibrates the hierarchy to physical distance.

### Usage 
```
python quantile_regression.py --q 0.2 --balance --outdir /path/to/output/folder --rf_pred_path /path/to/file/avg_allyPred.npy --sdir /path/to/output/folder/train_test_data --hierarchy_path /path/to/hierarchy.louvain.termStats --predicted_sim_path /path/to/predicted_resnik_sim.ddot
 ```

### Required arguments for quantile_regression.py

`--q` quantile for quantile regression model

`--outdir` Full path to the folder where results will be saved in with unique file identifier.

`--rf_pred_path` Full path to file with prefix avg_allyPred_modelfname_prefix.npy saved in pred_RF.py

`--sdir` Full path to the folder training and testing data was saved in format_random_forest.py 

`--hierarchy_path` Full path to file with hierarchy saved in community_detection.py with file extension louvain.termStats

`--predicted_sim_path` Full path to random forest predicted similarity matrix predicted_resnik_sim.ddot

### Optional arguments for quantile_regression.py

`--balance` 

