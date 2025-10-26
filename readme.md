# Cell Decoder: Decoding cell identity with multi-scale explainable deep learning

![Workflow](./resource/framework.png)

## Package: `celldecoder`

We created the python package called `celldecoder` that that decoding cell identity from gene expressions by explicitly modeling the multi-scale biological interactions, i.e., genes, pathways, and biological processes.

### Requirements

+ Python >= 3.8
+ torch == 2.0.1
+ torch-geometric == 2.3.1
+ CUDA 11.7

### Create environment

```
conda create -n celldecoder python=3.8
conda activate celldecoder
conda install pytorch=2.0.1 cudatoolkit=11.7 -c pytorch
pip install torch_geometric
pip install torch_scatter torch_sparse torch_cluster -f https://data.pyg.org/whl/torch-2.0.0+cu117.html
```

### Installation

The `celldecoder` python package is in the folder celldecoder. You can simply install it from the root of this repository using

```
pip install .
```

### Environment has been tested

`environment.yml`

## Usage

### Step 1: Constructing the model

#### Domain knowledge
Building the model requires the use of prior knowledge. Ensure the creation of 2 directories at the software's location: `/data/ppi` and `/data/reactome`. The corresponding data can be downloaded from https://figshare.com/articles/dataset/PPI_data/24921831.


+ `hierarchy`: gene-pathway mapping & hierarchy pathway information
+ `ppi`: protein-protein interactions network

```py
from celldecoder.data import interactions,hierarchy
import json
#hierarchy
n_layers = 3 # n layers of the model
reactome = hierarchy.hierarchy_layer(species='HSA') #HSA: human,MMU:mouse  
layers = reactome.get_layers(n_levels=n_layers)
ref_adata.uns['hierarchy'] = json.dumps(layers) 
query_adata.uns['hierarchy'] = json.dumps(layers)
#ppi
ref_adata = interactions.data_mapping_ppi(ref_adata,ppi_data) #ppi_data: ppi network
```

#### Input:

+ `data`: an `AnnData` object of reference data and query data （checkout reference and query have the same feature）
+ `ppi_data` : pre-prepared ppi networks data
  
  #### Domain knowledge files
+ `./data/ppi`: human & mouse processed ppi data
+ `./data/reactome`: hierarchy pathway information

### Step 2: Training the model

```py
import celldecoder
dataset = "./data/hBone/hBone_ref_adata.h5ad"
device_id = 1
log_dir = f"./log/{dataset}"
# Train the cell decoder using the specified dataset `dataset` on the device `device_id`,
# logging to the specified directory `log_dir`, and using the `cell_label` parameter to specify the cell type label.
celldecoder.train(dataset=dataset, device_id=device_id, log_dir=log_dir, cell_label="cell_type")
```

#### Input:

+ `dataset`: dataset name 
+ `log_dir` : logging directory
+ `device_id`: gpu device id

See other arguments in celldecoder/config.py

#### Output:

+ `./log_dir/args.json` : configuration file
+ `./log_dir/best.pth` : best checkpoint weights

See command output for validation metrics.

### Step 3: Prediect by the model

```py
import celldecoder
device_id = 1
log_dir = f"./log/hBone"
dataset = "./data/hBone/hBone_query_adata.h5ad"
fn_process = "processed-test"
predict_type = 'cell'
cells = celldecoder.predict(dataset = dataset, device_id = device_id ,log_dir = log_dir, fn_process = fn_process, predict_type = predict_type)
```

#### Input:

+ `dataset`: dataset name 
+ `log_dir` : logging directory
+ `device_id`: gpu device id

#### Output:

See command output for test metrics.

### Step 4: Output embeddings by the model

```py
import celldecoder
device_id = 1
log_dir = f"./log/hBone"
dataset = "./data/hBone/hBone_query_adata.h5ad"
fn_process = "processed-test"
celldecoder.embed(dataset = dataset, device_id = device_id ,log_dir = log_dir, out_embed = "output", fn_process = fn_process)
```

#### Input:

+ `dataset`: dataset name 
+ `log_dir` : logging directory
+ `device_id`: gpu device id

#### Output:

Embeddings of the cells

### Step 5: Output feature explanations by the model

```py
import celldecoder
device_id = 1
log_dir = f"./log/hBone"
dataset = "./data/hBone/hBone_query_adata.h5ad"
fn_process = "processed-test"
celldecoder.explain_feature(dataset = dataset, device_id = device_id ,log_dir = log_dir, explain_method = "grad", fn_process = fn_process)
```

#### Input:

+ `dataset`: dataset name 
+ `log_dir` : logging directory
+ `device_id`: gpu device id

#### Output:

Feature explanations of the cells

### Step 6: Output ppi explanations by the model

```py
import celldecoder
device_id = 1
log_dir = f"./log/hBone"
dataset = "./data/hBone/hBone_query_adata.h5ad"
fn_process = "processed-test"
exp_dict ={
    "correlation": 0,
    "multi_atten": 1,
    "train_sample_gt": 0,
    "ce_loss_gt": 0,
    "exp_train_epochs": 100,
    "exp_lr": 0.01,
}
celldecoder.explain_ppi(dataset = dataset, device_id = device_id ,log_dir = log_dir, fn_process = fn_process, **exp_dict)
```

#### Input:

+ `dataset`: dataset name 
+ `log_dir` : logging directory
+ `device_id`: gpu device id

#### Output:

PPI explanations of the cells

### Example Demo:

[Guided Tutorial](test/tutorial.ipynb)

### Cite Cell  Decoder:

Zhu, J., Zhang, Z., Xiang, Y., Xie, B., Dong, X., Xie, L., ... & Chang, C. (2025). Cell Decoder: decoding cell identity with multi-scale explainable deep learning. Genome Biology, 26(1), 359.

