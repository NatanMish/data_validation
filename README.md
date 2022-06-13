# Data Validation for Data Science

This repository contains data, code and Jupyter notebooks for the validation of data science 
projects tutorial. The tutorial consists of three sections for each step in the 
production data science model life cycle:
1. [Database management](notebooks/database_data_validation.ipynb) (using [Great Expectations](https://great-expectations.readthedocs.io/en/latest/))
2. [Training pipeline](notebooks/training_pipeline_data_validation.ipynb) (using [Pandera](https://pandera.readthedocs.io/en/stable/))
3. [Model serving](notebooks/model_serving_data_validation.ipynb) (using [Pydantic](https://pydantic-docs.helpmanual.io/))

Each section comes with a notebook in which there are explanations, code snippets and 
exercises.

## Data

Dataset used for the purposes of this tutorial is taken from the [House prices 
prediction competition on Kaggle](https://www.kaggle.com/competitions/home-data-for-ml-course/data). 
Two CSV files located in the `data` folder: `train.csv` and `test.csv`.

## Instructions

To Follow the notebooks and exercises there are two options:
1. Use your own Python environment with Jupyter installed. The notebooks are run using 
the `jupyter notebook` command, select the notebook you want to run in the `notebooks` 
folder and follow the instructions. **For running the different tools with all of the 
features available it is recommended to use `Python 3.8` and up**.
2. Use Google Colaboratory without any pre-installation needed. Click the link to go to 
the repository's [GitHub page](https://github.com/NatanMish/data_validation). Choose one
of the notebooks in the `notebooks` folder and from the interactive view, click on the 
link to `open in Colab`.


