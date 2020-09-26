# High Performance Jupyter: Faster workloads with Dask and RAPIDS

Code supporting JupyterCon 2020 talk "High performance Jupyter: Faster workloads with Dask and RAPIDS".

Slides are [here](slides.pdf)

# Summary

[Dask](https://dask.org/) is a parallel computing framework that scales from your laptop to a cluster of thousands of machines. [RAPIDS](http://rapids.ai/) is a GPU-computing framework that pushes traditional CPU workloads to the GPU. Dask and RAPIDS together allow you to scale both up and out! There are several notebooks in this repo that progressively tell the story of accelerating Jupyter with these two tools:

| Notebook                                     | Hardware       | Tools         | Data size, compute time | 
|----------------------------------------------|----------------|---------------|-------------------------|
| [laptop.ipynb](laptop.ipynb)                 | Laptop         | Pandas/Scikit | 1x, baseline 🔴            | 
| [dask.ipynb](dask.ipynb)                     | Laptop         | Dask          | 10x, slow 🟡               | 
| [dask-cluster.ipynb](dask-cluster.ipynb)     | Cluster (CPU)  | Dask          | 10x, fast 🟢               |
| [rapids.ipynb](rapids.ipynb)                 | GPU            | RAPIDS        | 1x, _super_ fast ⚡️⚡️        |
| [rapids-cluster.ipynb](rapids-cluster.ipynb) | Cluster (GPU)  | RAPIDS+Dask   | 10x, _super_ fast 🤯🤯        |

The [laptop.ipynb](laptop.ipynb) and [dask.ipynb](dask.ipynb) notebooks can be run on any machine that has >4GB RAM. The [rapids.ipynb](rapids.ipynb) notebook can be run on any machine with a CUDA-enabled GPU. The [dask-cluster.ipynb](dask-cluster.ipynb) and [rapids-cluster.ipynb](rapids-cluster.ipynb) need to be run on clusters of machines (most easily obtained by renting from a cloud provider).

# Performance comparisons

Here are some timing comparisons from the different notebooks. Please note that repeated experiments were not performed, and hardware specifications were different for each notebook. This is meant to serve as a rough overview of the speedups from the different tools. Also note that the multi-node environments would continue to show speed improvements by adding more machines.

(all times reported in seconds)

**Small data size (~7 million rows)**

| Task               | Single-node CPU (Pandas/Scikit) | Single GPU (RAPIDS) |
|--------------------|---------------------------------|---------------------|
| .read_csv()        | 80                              | 8.85                |
| .describe()        | 4.33                            | 0.82                |
| train_test_split() | 2.77                            | 0.64                |
| Random forest      | 149                             | 1.31                |

**Large data size (~85 million rows)**

| Task                     | CPU cluster (Dask) | GPU cluster (RAPIDS+Dask) |
|--------------------------|--------------------|---------------------------|
| Row count/size           | 70                 | 32.6                      |
| .describe()              | 48.7               |                           |
| Feature eng (persist())  | 40.5               | 17.6                      |
| Random forest            |                    | 3.82                      |

**Grid search (~300,000 rows)**

|             | Single-node CPU (Pandas/Scikit) | CPU cluster (Dask) |
|-------------|---------------------------------|--------------------|
| Grid search | 226                             | 17.2               |

# Setup

The `environment.yml` file has the necessary packages required to run the [laptop.ipynb](laptop.ipynb) and [dask.ipynb](dask.ipynb) notebooks. There are a couple commands necessary after creating the environment to initialize the Dask extension for JupyterLab, and then you can fire up JupyterLab!

```bash
conda env create -f environment.yml
conda activate dask

jupyter labextension install dask-labextension
jupyter serverextension enable dask_labextension

jupyter lab
```

### RAPIDS

RAPIDS requires a Linux OS and CUDA-enabled GPU. As such the installation will be different depending on your hardware. RAPIDS has a [handy guide here](https://rapids.ai/start.html) that gives you the `conda install` command to run! There is also a JupyterLab [extension for monitoring GPU usage](https://github.com/rapidsai/jupyterlab-nvdashboard) included with RAPIDS, so you can run a command to enable that.

```bash
conda activate dask
conda install ...  # command from RAPIDS guide

conda install -c conda-forge jupyterlab-nvdashboard
jupyter labextension install jupyterlab-nvdashboard

jupyter lab
```

### Running on Saturn Cloud

In the talk I utilize [Saturn Cloud](https://saturncloud.io) for the [rapids.ipynb](rapids.ipynb), [dask-cluster.ipynb](dask-cluster.ipynb), and [rapids-cluster.ipynb](rapids-cluster.ipynb) notebooks (disclosure: I work at Saturn Cloud). Saturn makes it easy to configure Python environments and launch machines (and clusters!) that support Dask and RAPIDS. You can get going pretty quickly with a [free trial on the Hosted version](https://www.saturncloud.io/s/plans/) and run the notebooks there.

There needs to be two separate Projects for the CPU (Dask) and GPU (RAPIDS) notebooks:

#### CPU (Dask) Jupyter

- Name: "dask"
- Size: "Large - 2 cores - 16 GB RAM"
- Image: "saturncloud/saturn:\*"


#### GPU (RAPIDS) Jupyter

- Name: "rapids"
- Size: "T4-XLarge - 4 cores - 16 GB RAM - 1 GPU"
- Image: "saturncloud/saturn-gpu:\*"
- Start script:
    ```bash
    conda install -c conda-forge -n base -y jupyterlab-nvdashboard
    jupyter labextension install jupyterlab-nvdashboard
    ```

Once you start the Jupyter server and jump into JupyterLab, open a new Terminal window to grab the code:

```bash
git clone https://github.com/rikturr/high-performance-jupyter /tmp/high-performance-jupyter
cp -r /tmp/high-performance-jupyter/* /home/jovyan/project/
```

The notebooks will take care of the rest!
