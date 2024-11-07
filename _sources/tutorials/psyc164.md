# Environment for PSYC164

## Environment setup

If you are taking the PSYC164 course and hope to use the resources on Discovery, you can use the conda environment we prepared for the class.

First set up the minoconda environment:

```bash
eval "$(/dartfs/rc/lab/D/DBIC/DBIC/f001693/singularity_home/psyc164/miniconda3/bin/conda shell.bash hook)"
```

Then activate the `nb` environment:

```bash
conda activate nb
```

## Linking the data directory

A copy of the data that will be used for the class is available at `/dartfs/rc/lab/D/DBIC/DBIC/f001693/singularity_home/psyc164/nb-data`. You can link this directory to your home directory by running the following command:
```bash
ln -s /dartfs/rc/lab/D/DBIC/DBIC/f001693/singularity_home/psyc164/nb-data $HOME/.neuroboros-data
```

If `$HOME/.neuroboros-data` already exists, you can remove it with the following command:
```bash
rm $HOME/.neuroboros-data
```

## Create the environment

For records, the environment was created using the following commands:

```bash
conda create --name nb "python>=3.9" pip
conda activate nb
pip install -U numpy scipy scikit-learn nibabel pandas matplotlib seaborn ipython jupyter jupyterlab nipy hyperalignment
pip install -U neuroboros
```

The environment might be updated as the course progresses.
To keep a copy of the environment as it currently is, you can [clone the environment](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#cloning-an-environment
).
