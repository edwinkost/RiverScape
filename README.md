# RiverScape


A set of Jupyter notebooks for the evaluation of river management measures.



## How to install

A few steps are required to run the Jupyter notebooks:

 1. You will need a working Python environment, we recommend to install Miniconda. Follow their instructions given at:

    [https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html)

 2. Download the [requirements file](https://github.com/UU-Hydro/RiverScape/blob/master/requirements.yaml) and use it to install all modules required to run the Jupyter notebooks:

    `conda env create -f requirements.yaml`

    The requirements file will create a Python environment named `riverscape`. In case you prefer a different name you need to edit the `requirements.yaml` file.

 3. Activate the environment in a command prompt:

    `conda activate riverscape`

 4. Clone or download this repository:

    `git clone https://github.com/UU-Hydro/RiverScape.git`

    This will clone RiverScape into the current working directory.

General information on Jupyter notebooks and manuals can be found [here](https://jupyter.readthedocs.io/en/latest/).

## How to run

Activate the environment in a command prompt:

`conda activate riverscape`

Change to the RiverScape `scripts` directory.
You can start Jupyter from the command prompt and afterwards select a notebook in your browser:

`jupyter-notebook`



You can also open individual notebooks directly by specifying the filename, e.g. the intervention planning with:

`jupyter-notebook intervent_parameter.ipynb`



## Available notebooks

The following Jupyter notebooks are provided:

  1. Intervention planning: [intervent_parameter.ipynb](scripts/intervent_parameter.ipynb)

  2. Evaluation of biodiversity: [biosafe.ipynb](scripts/biosafe.ipynb)

  3. Evaluation of implementation costs: [cost_evaluation.ipynb](scripts/cost_evaluation.ipynb)

  4. Evaluation of landowner involvement: [landowner_evaluation.ipynb](scripts/landowner_evaluation.ipynb)




## Exemplary output set of measures

In case you want to run the evaluation notebooks without explicitly defining your own set of measures first you can load output data from a pre-defined set of measures.
A single measure is included in the `output` folder.
An ensemble of measures is provided as compressed file.
Extract the file `example_measures_waal.zip` in the `outputs` folder.
You'll get a `example_measures_waal` subfolder containing outputs of 17 measures.



