# Using `neuroboros` on Discovery

## Accessing Discovery

### Dartmouth network or VPN

[Discovery](https://rc.dartmouth.edu/index.php/discovery-overview/) is the high-performance computing (HPC) cluster of Dartmouth College. 
To use Discovery, you need to be using either the Dartmouth network (eduroam) or the Dartmouth VPN (GlobalProtect).
Here are the instructions on how to setup the [eduroam network](https://services.dartmouth.edu/TDClient/1806/Portal/KB/ArticleDet?ID=64684) and the [GlobalProtect VPN](https://services.dartmouth.edu/TDClient/1806/Portal/KB/ArticleDet?ID=72395) on your computer.

### Discovery account

When you are using either eduroam or the VPN, you can create a Discovery account [here](https://dashboard.dartmouth.edu/research/hpc_account).

### Computing resources

You may have access to the [DBIC resources on Discovery](https://www.dartmouth.edu/dbic/research_infrastructure/discovery.html). Your lab may also has its own resources. Please ask your PI about the resources that you can use and how to access them.

### SSH access to Discovery

After you have created a Discovery account, you can access Discovery with SSH:
```bash
ssh NetID@discovery7
```
You need to replace `NetID` with your real NetID.
If you don't know your NetID, you can [look it up](https://lookup.dartmouth.edu/).

```{margin}
In addition to the config, you can also setup an [SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
```

You can edit your SSH config (located at `$HOME/.ssh/config`) to make things easier. For example, you can add the following lines to your SSH config (again, replace `NetID` with your real NetID):
```
Host d7
  HostName discovery7.dartmouth.edu
  User NetID
Host ndoli
  HostName ndoli.dartmouth.edu
  User NetID
```

With these settings, you can access Discovery much easier:
```bash
ssh d7
```

## Linking the data directory

```{margin}
It is possible mount DartFS on your computer, like an external hard drive. See [here](https://rc.dartmouth.edu/index.php/dartfs-access-guide/) for more information.
```

If you are using `neuroboros` on Discovery, you can take advantage of the datasets that are already available on the cluster through [DartFS](https://rc.dartmouth.edu/index.php/draft/data-storage-dartfs/). What you need to do is to create a symbolic link to the data directory in your home directory:

```bash
ln -s /dartfs/rc/lab/H/HaxbyLab/feilong/nb-data $HOME/.neuroboros-data
```

## Setting up the Python environment

### Loading the Python module
The Python module is already installed on Discovery. You can load the Python module with the following command:
```bash
module load python
```

To automatically load the Python module when you log in, you can add add it to the list of auto-loaded modules:
```bash
module initadd python
```

### Simple installation

`neuroboros` can be installed with `pip`:
```bash
python -m pip install -U neuroboros
```

If you encounter any permission issues, you can install `neuroboros` with the `--user` option. This will install `neuroboros` in your home directory:
```bash
python -m pip install --user -U neuroboros
```

### Virtual environment

Ideally, you should create a virtual environment for each project of yours. You can create a virtual environment with Anaconda and the yml config file:
```bash
conda env create -f nb.yml
```

The yml config file is available [here](nb.yml).

This will create a virtual environment named `nb`. You can activate the virtual environment with the following command:
```bash
conda activate nb
```

Alternatively, you can create a virtual environment with Python's [`venv` module](https://docs.python.org/3/library/venv.html).

## Submitting an interactive job

Currently, Discovery uses the SLURM job submission system. To test your job on Discovery, you can submit an interactive job with the following command:
```bash
srun --nodes=1 --ntasks-per-node=1 --mem=8GB --cpus-per-task=1 \
    -t 24:00:00 -A DBIC --pty /bin/bash
```

This command requires 1 CPU, 8GB memory, and 24 hours of walltime. You can change these parameters according to your needs.
If you have a different account other than DBIC, you need to change the `-A` parameter accordingly.

### Time and memory usage

When you test your script using an interactive job, make sure that you record the execution time and memory usage of your script. This will help you to decide the required walltime and memory for your actual job. You can use the `time` command to record the execution time of your script:
```bash
/bin/time -v python my_script.py
```

### Converting a Jupyter notebook to a Python script

:::{margin}
If the notebook's filename is `my_script.ipynb` as in the example, the converted Python script will be saved as `my_script.py`.
:::
If your script is a Jupyter notebook, you can convert it to a Python script with the following command:
```bash
jupyter nbconvert --to python my_script.ipynb
```

## Submitting a batch job

When you are ready to submit your job, you can write a job submission script. Here is an example job submission script:

```bash
#!/bin/bash

#SBATCH --job-name=jobname
#SBATCH -A DBIC
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=8G
#SBATCH --time=24:00:00
#SBATCH --partition=standard
#SBATCH -o logs/%A_%a_%j.out
#SBATCH -e logs/%A_%a_%j.err
#SBATCH --nice=999

# Set up the Python environment
module unload python
__conda_setup="$('/optnfs/python/el7/3.7-Anaconda/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/optnfs/python/el7/3.7-Anaconda/etc/profile.d/conda.sh" ]; then
        . "/optnfs/python/el7/3.7-Anaconda/etc/profile.d/conda.sh"
    else
        export PATH="/optnfs/python/el7/3.7-Anaconda/bin:$PATH"
    fi
fi
unset __conda_setup
conda activate tc

# Run the script
python my_script.py
```

Suppose the job submission script is saved as `submit.sh`. You can submit the job with the following command:
```bash
sbatch submit.sh
```

When you submit the job, make sure the log directory exists. In the example above, the log directory is `logs/`. You can create the log directory with the following command:
```bash
mkdir logs
```

If you want to know more about the SLURM job submission system, you can check out the [SLURM documentation](https://slurm.schedmd.com/documentation.html).

## Submitting an array of jobs

When you need to run multiple jobs, you can change the job submission script to submit an array of jobs. You just need to add the following line to the job submission script:
```bash
#SBATCH --array=0-9
```
or add the parameter to the `sbatch` command:
```
sbatch --array=0-9 submit.sh
```

This will submit 10 jobs with the same script, where each job has a different job array index from 0 to 9. The job array index is stored in the environment variable `SLURM_ARRAY_TASK_ID`. You can use this variable as an input to your Python script. For example
```bash
python my_script.py $SLURM_ARRAY_TASK_ID
```

:::{margin}
See [here](https://docs.python.org/3/library/sys.html#sys.argv) on how to access command-line arguments in Python using `sys.argv`.
:::
In your Python script, you can use `sys.argv[1]` to access the job array index (in this case `sys.argv[0]` will be `my_script.py`). For example
```python
import sys
arg = int(sys.argv[1])
print(arg)
```
