[PR]: https://github.com/cavalab/srbench/compare/Competition2022...change-to-my-fork
[example]: https://github.com/cavalab/srbench/blob/Competition2022/submission/feat-example/
[metadata]: https://github.com/cavalab/srbench/blob/Competition2022/submission/feat-example/metadata.yml
[regressor]: https://github.com/cavalab/srbench/blob/Competition2022/submission/feat-example/regressor.py
[env]: https://github.com/cavalab/srbench/blob/Competition2022/submission/feat-example/environment.yml
[paper]: https://datasets-benchmarks-proceedings.neurips.cc/paper/2021/hash/c0c7c76d30bd3dcaefc96f40275bdc0a-Abstract-round1.html
[branch]: https://github.com/cavalab/srbench/tree/Competition2022

# SRBench 2022 Competition Guidelines
To participate, the steps are relatively straightforward. 
Participants fork this repo, add a method in the submission folder (see [submission/feat-example](https://github.com/cavalab/srbench/blob/Competition2022/submission/feat-example/)), and submit it as a [pull request][PR] to the [Competition2022 branch][branch]. 
Once submitted, the continuous integration process will give feedback if there are any problems with the submission. 
Participants can then update their PR as necessary to have it pass the tests. 

Once everything is working, participants can basically sit tight: the competition organizers will then set about testing the methods on a variey of datasets. 

## Instructions

1. Fork this repository and clone it. Check out the [Competition2022 branch][branch].

    ```bash
    git clone git@github.com:your-name/srbench
    cd srbench
    git checkout Competition2022 # may need to "git fetch" first if this fails
    ```

2. Make a folder in the `submission/` named after your method. You can start with a template by copying the [`submission/feat-example`][example] folder, renaming and editing it. 

3. In the folder, put the contents of your submission. This folder should contain:

    1. `metadata.yml` (**required**): A file describing your submission, following the descriptions in [submission/feat-example/metadata.yml][metadata]. 
    2. `regressor.py` (**required**): a Python file that defines your method, named appropriately. See [submission/feat-example/regressor.py][regressor] for complete documentation. 
        It should contain:
        -   `est`: a sklearn-compatible `Regressor` object. 
        -   `model(est, X=None)`: a function that returns a [**sympy-compatible**](https://www.sympy.org) string specifying the final model. It can optionally take the training data as an input argument. See [guidance below](###-returning-a-sympy-compatible-model-string). 
        -   `eval_kwargs` (optional): a dictionary that can specify method-specific arguments to `evaluate_model.py`.
        
    3. `LICENSE` *(optional)* A license file
    4. `environment.yml` *(optional)*: a [conda environment file](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file) that specifies dependencies for your submission. 
    See [submission/feat-example/environment.yml][env] for an example.
    It will be used to update the baseline environment (`environment.yml` in the root directory). 
    To the extent possible, conda should be used to specify the dependencies you need. 
    If your method is part of conda, great! You can just put that in here and leave `install.sh` blank. 
    5. `install.sh` *(optional)*: a bash script that installs your method. 
    **Note: scripts should not require sudo permissions. The library and include paths should be directed to conda environment; the environmental variable `$CONDA_PREFIX` specifies the path to the environment.
    6. additional files *(optional)*: you may include a folder containing the code for your method in the submission. Otherwise, `install.sh` should pull the source code remotely. 

    7. Commit your changes and submit your branch as a [pull request Competition 2022 branch][PR]. 

    8. Once the tests pass, you will be an official competitor!

### Version control

The install process should guarantee that the version for your algorithm that gets installed is **fixed**. 
You can do this in many ways: 

1. Including the source code with the submission. 
2. Pulling a tagged version/release of your code from a git repository. 
3. Checking out a specific commit, as in the provided example. 

## Regressor Guide

For detailed documentation, carefully read the example `regressor.py` file in [submission/feat-example/regressor.py][regressor], which describes several options for configuring your algorithm. 

### Returning a sympy compatible model string

In order to check for exact solutions to problems with known, ground-truth models, each SR method returns a model string that can be manipulated in [sympy](https://www.sympy.org). 
Assure the returned model meets these requirements:

1. The variable names appearing in the model are identical to those in the training data, `X`, which is a `pd.Dataframe`. 
If your method names variables some other way, e.g. `[x_0 ... x_m]`, you can
specify a mapping in the `model` function such as:

```python
def model(est, X):
    mapping = {'x_'+str(i):k for i,k in enumerate(X.columns)}
    new_model = est.model_
    for k,v in mapping.items():
        new_model = new_model.replace(k,v)
```

2. The operators/functions in the model are available in [sympy's function set](https://docs.sympy.org/latest/modules/functions/index.html). 

**Note:** In part of the competition (see the [judging criteria](JudgingCriteria.md)), models will be checked for symbolic equivalence with ground-truth formulae. 
If your method uses protected operators (e.g., `my_prot_div(a,b)= a/b if b!=0 else 1` or `my_prot_log(a)=log(abs(a)+eps)`), replace these with non-protected (sympy-compatible) versions when calling `model`. For example:

```python
def model(est, X):
    ... # any previous pre-processing
    new_model = new_model.replace("my_prot_div","/").replace("my_prot_log","log")
```

### CLI methods

Is your SR method typically called via a command line interface? 
Check out this [gist](https://gist.github.com/folivetti/609bc9b854c51968ef90aa675ccaa60d) to make a sklearn interface. 

## Competition Details

Once a method is successfully added to the Competition2022 branch, the organizing team will set about running each participant's method through a set of datasets. 
These datasets include synthetic and real-world tasks. 
Each dataset will have less than 10,000 samples and fewer than 100 features. 

### Stages

The competition consists of three stages, summarized in the table below. 

| Stage     | Qualification  | Synthetic | Real-World |
|---|---|---|---|
| Benchmark Data | PMLB-20 | Synthetic | Real-World |
| Criteria  | Better than linear regresion | Accuracy, Simplicity, Exact Solutions | Accuracy, Simplicity, Expert Assessment |

- The first stage is a simple filter: submitted methods must be better than a linear model on PMLB-20. 
PMLB-20 is a set of 20 datasets we will select from the [Penn Machine Learning Benchmark](https://github.com/EpistasisLab/pmlb) used in our [original SRBench analysis](https://scikit-learn.org/stable/modules/classes.html#hyper-parameter-optimizers). (Participants are free to tune on this benchmark.)
- The second and third stages are separate and independent tracks (a method can, in principle, win both): the *Synthetic* track and the *Real-World* track.
The judging criteria for these tracks are different but share some aspects.
For example, common judging criteria are accuracy (in terms of [R2 Score](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.r2_score.html?highlight=r2_score#sklearn.metrics.r2_score)) and simplicity (in terms of number of model components).

### Judging Criteria

Full details on the judging criteria can be found [here](JudgingCriteria.md).

### Computing Environment

Experiments will be run in a heterogeneous cluster computing environment composed of several hosts with 24-28 core Intel(R) Xeon(R) CPU E5-2690 v4 @ 2.60GHz processors and 250 GB of RAM. 

The cluster uses an LSF scheduler. 
We define a _job_ as the training of an algorithm on a single dataset. 
For each job, an algorithm will have access to the following resources:

| Benchmark | PMLB-20 | Synthetic | Real-World |
|-----------|--------:|----------:|-----------:|
| RAM       | 16 GB   | 16 GB     | 16 GB      |
| CPU Cores | 4       | 8         | 8          |
| Wall-clock Time (H) | 24 | 1-10 | 10 |


CPU Cores for each job will span a single host, so methods are encouraged to support CPU-level parallelism. 



### Time Budget

Methods must adhere to a fixed time budget for the competition. 
*These time budgets are designed to be **generous**, not **restrictive***. 
Our goal is to allow "slow" implementations enough time to learn. 

All datasets will be less than 10,000 rows and fewer than 100 features. 
The time limits are as follows:

- For datasets up to 1000 rows, 60 minutes (1 hour)
- For datasets up to 10000 rows, 600 minutes (10 hours)


If a call to `est.fit()` takes longer than the alotted time, it will receive
a SIGALRM signal and be terminated. Users may choose to handle this signal if
they wish in order to return an "any time" solution. 

**The preferred approach is that participants design their algorithms to converge in less than an hour for 1000x100 datasets, and less than 10 hours for 10000x100 datasets.**
Participants are also encouraged to include a `max_time` parameter and set it appropriately. 

To define dataset-specific runtime parameters, users can define a `pre_train()`
function in `regressor.py` as part of `eval_kwargs`. 
As an example, one could define the following in `regressor.py`:

```python
def pre_train_fn(est, X, y): 
    """set max_time in seconds based on length of X."""
    if len(X)<=1000:
        max_time = 360 
    else:
        max_time = 3600
    est.set_params(max_time)

# pass the function to eval_kwargs
eval_kwargs = {
    'pre_train': pre_train_fn
}
```

### Hyperparameter Tuning

**Warning** If choose to conduct hyperparameter tuning, this counts towards the time limit. 
In the example below we show one way to set the `max_time` accordingly.  
{: .notice--warning}

Unlike our [prior study][paper], the competition does not automatically conduct hyper-parameter tuning for each method. 
Thus, it is up to participants to decide whether they want to include hyperparameter tuning as part of algorithm training in `regressor.py`. 
Here is an example of how to do so:


```python
from sklearn.model_selection import GridSearchCV
from mylibrary import MyMethod

hyper_params = { 'alpha': (1e-04,0.001,0.01,0.1,1,) }

# define your base estimator
base_est=MyMethod() 

# set est to be a GridSearchCV estimator
est = GridSearchCV(estimator=base_est, 
                   param_grid=hyper_params, 
                   cv=5,
                   refit=True)

# set max time w.r.t. hyper-param tuning
num_combos = 5  # five values of alpha
num_cv = 5      # 5 cross-validation splits as above
total_runs = num_combos * num_cv + 1 # +1 because refit=True

def pre_train_fn(est, X, y): 
    """set max_time in seconds based on length of X & hyper-parameter tuning"""
    if len(X)<=1000:
        # account for hyper-parameter tuning, remove 1 extra second of slack to be sure it will terminate on time
        max_time = 360 // total_runs - 1 
    else:
        max_time = 3600 // total_runs - 1
    est.set_params(max_time)

# pass the function eval_kwargs
eval_kwargs = {
    'pre_train': pre_train_fn
}

# additional definitions
# ...
```

In this example, the estimator is a GridSearchCV object wrapping a custom method. 
During fit, it will tune the `alpha` parameter using cross-validation. 
Moreover, the `pre_train_fn` is set to run the method so that the time limit will not be exceeded.
For other hyper-parameter optimizers than GridSearchCV, see, e.g., the [scikit-learn docs](https://scikit-learn.org/stable/modules/classes.html#hyper-parameter-optimizers).