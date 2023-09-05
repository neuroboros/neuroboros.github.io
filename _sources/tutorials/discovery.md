# Using `neuroboros` on Discovery

## Linking the data directory

[Discovery](https://rc.dartmouth.edu/index.php/discovery-overview/) is the high-performance computing (HPC) cluster of Dartmouth College. If you are using `neuroboros` on Discovery, you can take advantage of the datasets that are already available on the cluster through [DartFS](https://rc.dartmouth.edu/index.php/draft/data-storage-dartfs/). What you need to do is to create a symbolic link to the data directory in your home directory:

```bash
ln -s /dartfs/rc/lab/H/HaxbyLab/feilong/nb-data $HOME/.neuroboros-data
```

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

