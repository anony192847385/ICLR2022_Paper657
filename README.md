# Efficient Neural Causal Discovery without Acyclicity Constraints

[![Open In Collab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/anony192847385/ICLR2022_Paper657/blob/main/walkthrough.ipynb) 

This is the anonymized code for the ICLR 2022 submission "Efficient Neural Causal Discovery without Acyclicity Constraints".

## Requirements

The code is written in PyTorch (1.9) and Python 3.8. Higher versions of PyTorch and Python are expected to work as well.

We recommend to use conda for installing the requirements. If you haven't installed conda yet, you can find instructions [here](https://www.anaconda.com/products/individual). The steps for installing the requirements are:

1. Create a new environment from the provided YAML file:
   ```setup
   conda env create -f environment.yml
   ```
   The environment installs PyTorch with CUDA 11.1. Adjust the CUDA version if you want to install it with CUDA 10.2, or remove it from the environment file if you want to install it on a CPU-only system.
   
2. Activate the environment
   ```setup
   conda activate enco
   ```

### Datasets

To reproduce the experiments in the paper, we provide datasets of causal graphs for the synthetic, confounder, and real-world experiments. The datasets can be download by executing `download_datasets.sh` (requires approx. 600MB disk space). Alternatively, the datasets can be accessed through [this link](https://drive.google.com/file/d/1mJXJpvkG8Ol4w6QlbzW4EETjpXmHPlMX/view?usp=sharing) (unzip the file in the `causal_graphs` folder).

## Running experiments

The repository is structured in three main folders:
* `causal_graphs` contains all utilities for creating, visualizing and handling causal graphs that we experiment on.
* `causal_discovery` contains the code of ENCO for structure learning.
* `experiments` contains all utilities to run experiments with ENCO on various causal graphs.

Details on running experiments as well as the commands for reproducing the experiments in the paper can be found in the [`experiments`](experiments/) folder.

### Simple example

We created a quick walkthrough tutorial that goes through the most important functions/components in the repository in [`walkthrough.ipynb`](walkthrough.ipynb). In short, ENCO can be applied as follows:

```python
from causal_graphs.graph_generation import generate_categorical_graph, get_graph_func  # Functions for generating new graphs
from causal_discovery.enco import ENCO

# Create a graph on which ENCO should be applied
graph = generate_categorical_graph(num_vars=8, 
                                   min_categs=10,
                                   max_categs=10,
                                   graph_func=get_graph_func('random'),
                                   edge_prob=0.4,
                                   seed=42)

# Create ENCO object
enco_module = ENCO(graph=graph)
if torch.cuda.is_available():
    enco_module.to(torch.device('cuda:0'))

# Run causal discovery
predicted_adj_matrix = enco_module.discover_graph(num_epochs=2)
```

## FAQ

<details>
<summary>How is the repository structured?</summary>
<br>

We give a quick walkthrough of the most important functions/components in the repository in [`walkthrough.ipynb`](walkthrough.ipynb).  

</details>

<details>
<summary>Can I also run the experiments on my CPU?</summary>
<br>

Yes, a GPU is not a strict constraint to run ENCO. Especially for small graphs (about 10 variables), ENCO is similarly fast on a multi-core CPU than on a GPU. To speed up experiments for small graphs on a CPU, it is recommended to reduce the hidden size from `64` to `32`, and the graph samples in graph fitting from `100` to `20`.  

</details>

<details>
<summary>How can I apply ENCO to my own dataset?</summary>
<br>

If your causal graph/dataset is specified in a `.bif` format as the real-world graphs, you can directly start an experiment on it using `experiments/run_exported_graphs.py`. The alternative format is a `.npz` file which contains a observational and interventional dataset. The file needs to contain the following keys:
   
* `data_obs`: A dataset of observational samples. This array must be of shape [M, num_vars] where M is the number of data points. For categorical data, it should be any integer data type (e.g. np.int32 or np.uint8).
* `data_int`: A dataset of interventional samples. This array must be of shape [num_vars, K, num_vars] where K is the number of data points per intervention. The first axis indicates the variables on which has been intervened to gain this dataset.
* `adj_matrix`: The ground truth adjacency matrix of the graph (shape [num_vars, num_vars], type bool or integer). The matrix is used to determine metrics like SHD during/after training. If the ground truth matrix is not known, you can submit a zero-matrix (keep in mind that the metrics cannot be used in this case).

</details>

<details>
<summary>Can I apply ENCO to continuous or categorical data?</summary>
<br>

Both data types are supported in this repository. Simply make sure that the numpy array has the data type `np.float32` for continuous experiments, and `np.uint8` or `np.int32` for categorical data.  

</details>
