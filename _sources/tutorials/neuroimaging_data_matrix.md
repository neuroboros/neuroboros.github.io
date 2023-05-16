---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Neuroimaging data as a matrix

+++

In this Jupyter Notebook we demonstrate how to access neuroimaging data with the `neuroboros` package, and what the data looks like.

```{code-cell} python3
import numpy as np
import neuroboros as nb
import matplotlib.pyplot as plt
```

Datasets can be accessed using the name of the dataset.
For example, `nb.Forrest` is the [3 T data](https://doi.org/10.1038/sdata.2016.92) of the [StudyForrest](https://www.studyforrest.org) dataset.

For most of the datasets, `dset.subjects` is a list of subject IDs.

```{code-cell} python3
dset = nb.Forrest()
sids = dset.subjects
print(type(sids), len(sids))
print(sids[:3])
```

`dset.get_data` is the method used to load the data files and preprocess them.
Four positional parameters are mandatery, that is, `sid` (subject ID), `task` (name of the task), `run` (run number, starting from 1), and `lr` ('l' for left hemisphere, 'r' for right, 'lr' for both).

```{code-cell} python3
sid = sids[0]
dm = dset.get_data(sid, 'forrest', 1, 'l')
```

The data is stored in a 2-D NumPy array, usually referred to as a data matrix.
The first dimension is the time points, and the second dimension is the vertices.
This data matrix is the responses of the left hemisphere to first run of the movie, and it has 451 time points and 9675 vertices.

+++

Sometimes the first dimension can be stimuli, and the second dimension can be voxels or components.
These two dimensions are often referred to as samples and features in machine learning contexts.

```{code-cell} python3
print(f'{type(dm)}, {dm.dtype}')
print(dm.shape)
```

Each column of the data matrix is the response time series of a cortical vertex.

```{code-cell} ipython3
fig, ax = plt.subplots(1, 1, figsize=(4, 1), dpi=200)
ax.plot(dm[:, 0])
ax.set_xlabel('Time points (TR)')
plt.show()
```

Each row of the data matrix is the spatial response pattern at a certain time point of the movie.

```{code-cell} ipython3
nb.plot([dm[0], dset.get_data(sid, 'forrest', 1, 'r')[0]],
       cmap='bwr', vmax=4, vmin=-4, title='Response pattern', width=600)
```

When we want to analyze a certain brain region, we can create a smaller data matrix using the indices of vertices in the region.

For example, `nb.sls` returns the vertex indices of each searchlight as a NumPy array.

```{code-cell} ipython3
sls = nb.sls('l', radius=20)
type(sls), len(sls)
```

```{code-cell} ipython3
type(sls[0]), len(sls[0])
```

`sls[0]` is the vertex indices of the searchlight around vertex 0, and of course, vertex 0 itself is in the searchlight.

```{code-cell} ipython3
sls[0]
```

The searchlight has 119 vertices, and by indexing `dm` with `sls[0]`, we created a new data matrix with 119 vertices.

```{code-cell} ipython3
sl_dm = dm[:, sls[0]]
sl_dm.shape
```
