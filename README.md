# MiSDEED: Microbial Synthetic Data Engine for Experimental Design
Microbiome data simulator for power analysis

<!-- generator schematic goes here -->
<!-- example synthetic data goes here -->

# Misdeed command-line interface

## Example
```bash
git clone https://github.com/pchlenski/misdeed
cd misdeed

# install requirements
pip install -r requirements.txt

# generate 10 trajectories
./misdeed -n 10 10,15 ./output
```

## Dependencies
Dependencies are listed in `requirements.txt` and can be installed using
```bash
pip install -r requirements.txt
```

# OmicsGenerator Python package

The `OmicsGenerator` class has the following public methods:
* `add_node`: Adds nodes to generator object.
* `add_interaction`: Adds interactions to generator object.
* `add_intervention`:  Adds an intervention to generator.
* `set_initial_value`: Sets a node value or growth rate.
* `get`: Gets a (node/interaction/intervention) by name.
* `remove`: Removes a node, intervention, or interaction from the generator by name.
* `generate`: Generates a single timecourse of synthetic data.
* `generate_multiple`: Generates several timecourses of synthetic data.
* `case_control`: Generates synthetic case and control timecourses.
* `copy`: Makes a deep copy of generator.
* `save`: Saves generator outputs as plaintext files.

In addition, the package contains a number of utility functions:
* `inference.infer_glv_params`: Infers community matrix, growth rates, and perturbation responses from absolute abundance data.
* `visualization.plot_timecourse`: Plots timecourses of compositional data.
* `visualization.plot_pca`: Plots PCA-transformed timecourses.

## Examples
The corresponding Jupyter notebook can be found at `notebooks/examples.ipynb`.

### Initialize generator
```python
# initialize generator:
gen = OmicsGenerator(
    100,                   # 100 time points
    ['mgx', 'mbx'],        # 2 nodes named 'mgx' and 'mbx'
    [15, 15],              # each node has 15 dimensions
    init_full=True         # set interaction matrices and growth rates randomly
)

# add intervention:
gen.add_intervention(
    'int1',                # intervention name
    'mgx',                 # apply to 'mgx' node
    10*np.random.rand(15), # set intervention response vector randomly
    start=50,              # start at t=50
    end=100                # go to end
)

```

### Single timecourse
```python
# run generator and plot:
x1, y1, z1 = gen.generate(dt=1e-2)
plot_timecourse(y1['mgx'])
plt.vlines(50, 0, 1)
```
![Single timecourse](./img/ex1.png)

### Multiple timecourses
```python
# run multi-generator and plot:
x2, y2, z2 = gen.generate_multiple(20)
plot_pca([y2], 'mgx')
```
![Multiple timecourses](./img/ex2.png)

### Case-control
```python
# run case-control and plot:
x3_control, y3_control, z3_control, x3_case, y3_case, z3_case = gen.case_control(100, .75, 'mgx', 1)
plot_pca([y3_control, y3_case], 'mgx', colors=['red', 'blue'], plot_trajectories=False)
```
![Case-control](./img/ex3.png)

### Using learned parameters
```python
# run case-control and plot:
gen4 = OmicsGenerator(
    100,                   # 100 time points
    ['mgx'],               # 1 nodes named 'mgx'
    [15],                  # 'mgx' has 8 dimensions
    init_full=True         # set interaction matrices and growth rates randomly
)

# add intervention:
gen4.add_intervention(
    'int1',                # intervention name
    'mgx',                 # apply to 'mgx' node
    10*np.random.rand(15), # set intervention response vector randomly
    start=50,              # start at t=50
    end=100                # go to end
)

# generate training data
x4, y4, z4 = gen4.generate(dt=1e-2)

# infer params
M, u, E = infer_glv_params(
    z4['mgx'],             # use (latent) absolute abundances
    gen4.get('int1').U,    # assume intervention indicator is known
    interaction_reg=100,   # L1 penalty for interaction matrix
    growth_reg=0,          # L1 penalty for growth rates
    intervention_reg=0,    # L1 penalty for intervention responses
    dt=1e-2                # same time-step as generator
)

# build inferred generator
gen4_inferred = OmicsGenerator(100, ['mgx'], [15])
gen4_inferred.add_interaction('M', 'mgx', 'mgx', M)
gen4_inferred.add_intervention('int1', 'mgx', E.reshape(-1), start=50, end=100)
gen4_inferred.set_initial_value('mgx', u.reshape(-1), growth_rate=True)
gen4_inferred.set_initial_value('mgx', gen4.get('mgx').initial_value) # same init. abundances as gen 1

x4_inferred, y4_inferred, z4_inferred = gen4_inferred.generate(dt=1e-2)

# plot both trajectories
colors = cm.get_cmap('magma', 20)(range(15))

for i in range(15):
    plt.plot(np.cumsum(z4_inferred['mgx'], axis=1)[:,i], alpha=0.5, c=colors[i])
    plt.plot(np.cumsum(z4['mgx'], axis=1)[:,i], alpha=0.5, c=colors[i])
```
![Using learned parameters](./img/ex4.png)


## Citation
TODO
