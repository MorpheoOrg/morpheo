# Training on Morpheo 


## Definition of Training tasks 

The Morpheo platform can handle different machine learning problems. 
Problems are defined by members of the Dreemcare project.  
The first problem addressed by the platform is sleep stages classification, other potential problems are sleep apnea detection, insomnia detection. 
The definition of a problem requires the creation of a `problem workflow` and a set of data with corresponding targets.  
A `problem workflow` mainly defines what are the **data targets** and the **performance metric** used to evaluate machine learning models.
A new `problem` must be registered in the `Orchestrator` by specifying the UUID of the `problem workflow` and other parameters, such as UUIDs of test data (see in [Details on Problem](#problem)). 

For a given problem, different algorithms can be submitted. 
The submission is made through `Analytics` and managed by the `Orchestrator`, which registers the new algorithm and creates associated training tasks. 
We call an `algorithm` a problem solution that has not been trained, and a `model` when this algorithm has been trained.   
**Training tasks are defined in two cases**:  
- when an algorithm is submitted.  
- when new data are submitted, models are updated.   
It implies that **only algorithms supporting online learning** can be submitted to the platform.   

 
Training tasks are specified by the `Orchestrator` with the definition of a `learnuplet` (See in [Details on Learnuplet](#learnuplet)), and are consumed by `Compute`.
`Compute` does the training based on the `problem workflow`, saves the resulting `model` after encryption in `Storage`, and sends the performances to the `Orchestrator`.  

Performances are computed on a **test dataset, fixed for a given problem and not accessible**.  
For now, no cross-validation is done and the **performance is public** (no public and private leaderboard). 
This choice is motivated by the fact that the platform is data-oriented: each time new data is uploaded, models are updated, which limit overfitting possibilities. 
This might change latter if needed.

Performances are also computed on each train data and send by `Compute` to the `Orchestrator` as feedback on the training for `Analytics`. 

### Training hypothesis summary

- model training at algorithm submission and at new data upload 
- submission of algortihms supporting online learning only  
- no cross-validation  
- model performance transparency (no public and private leaderboard)  

### <a name="problem"></a> Details on Problem

Problems are defined by a `problem` in the database of the `Orchestrator`. A `problem` contains the following elements:
- `uuid`: a unique identifier of the problem  
- `workflow`: UUID of the associated `problem workflow`.  
A `problem workflow` mainly defines what are the **data targets** and the **performance metric** used to evaluate machine learning models. 
An example of a `problem workflow` is given for sleep stages classification [here](https://github.com/MorpheoOrg/hypnogram-wf).
- `timestamp_upload`: timestamp of the problem creation.  
- `test_dataset`: list of UUIDs of test data, which are not accessible, except by `Compute`to compute performances of submitted algorithms.
- `size_train_dataset`: size of mini-batch for each training task. 

### <a name="learnuplet"></a> Details on Learnuplet

A learning task is defined by a `learnuplet`. A `learnuplet` is constructed by the `Orchestrator` in two cases: 
- when new data is uploaded  
- when a new algorithm is uploaded 
It is then used by `Compute` to do the training.

A learnuplet is made of the following elements:
- `uuid`: a unique identifier of the task  
- `problem`: the UUID of the problem associated to the learning task. 
- `train_data`: list of train data UUIDs, on which the learning will be done.  
- `test_data`: list of test data UUIDs, on which the performance of the algorithm is computed.
- `algo`: UUID of submitted algorithm.
- `model_start`: UUID of model to be trained. If `rank=0`, this UUID is the same as `algo`.
- `model_end`: UUID of the model obtained after training of `model_start`.  
- `rank`: rank of the task, which defines the order in which learnuplets must be trained. See in [construction of a learnuplet](#learnuplet_construction) for more details.   
- `worker`: UUID of worker which is in charge of the training task defined by this learnuplet.  
- `status`: status of the training task. It can be `waiting` if we are waiting for a model training with a lower rank, `todo` if the traiing job can start, `pending` if a worker is currently consuming the task, or `done` if training has been done successfully, or `failed`is trainig has been unsuccesfully done.  
- `train_perf`: list of performances on train data.  
- `perf`: performance on test data.  
- `training_creation`: timestamp of the learnuplet creation.
- `training_done`: timestamp of feeback from compute (when updating `status` to `done` or `failed`).

### <a name="learnuplet_construction"></a> Details on the construction of a learnuplet at algorithm upload

When uploading a new algorithm, its training is specified in `learnuplets` by the `Orchestrator`.  

For now, they are constructed following these steps:  
1. selection of associated `active data`: for now all data corresponding to the same problem with targets.  
This might change later to lower computational costs.  
2. for each mini-batch containing `size_train_dataset` (parameter fixed for the `problem`), creation of a learnuplet.   
The first learnuplet has `rank=0` and `status=todo`, and other have incremental values of `rank` and `status=waiting`. 
Each learnuplet contains the UUID of the model from which to start the training and UUID where to save the model after training.   
Model from which to start the learning is not defined for learnuplets with `status=waiting` (and `rank=i`) at learnuplet creation, but when `performance` of `learnuplet` with `rank=i-1` is registered on the `Orchestrator`. At this moment, the `Orchestrator` looks for the `model_end` of the `learnuplet` with the best performance to choose it as the `model_start` for learnuplet of `rank=i`, which `status` is now `todo`.

### Details on the construction of a learnuplet at data upload

When uploading new data, relevant models are updated.  

For now, the construction of corresponding `learnuplets` is made as follows:  
1. selection of relevant models called `active models`: for now all models corresponding to the same problem.  
This might change later to lower computational costs.  
2. for each algorithm: 
  - 2.1 find the model which has the best performance (which is not necessarily the one with the highest rank).
  - 2.2 for each mini-batch containing `size_train_dataset` (parameter fixed for the `problem`), creation of a learnuplet starting from the model found in 2.1.  

## Algorithm submission

Algorithms submitted to the platform must support **online learning**, since models are updated each time new data are uploaded.  
This is the case for all **neural networks**.  
Algorithms implemented in the package `scikit learn` can be used if they offer a `partial_fit` method (or a `warm_start` option, TODO: check...).

A submission corresponds to the submission of a [Docker](https://docs.docker.com/engine/getstarted/). It must contain the following elements:  
- a `train.sh` 


An example of a submission is given for sleep stages classification [here](https://github.com/MorpheoOrg/hypnogram-wf).

#### Note about saving trained models:  
Saving format is let free for now, since it is part of the code submitted for an algorithm.
- for neural networks, best solution is to save them in h5 files
- for sklearn models, avoid using [Pickle](http://scikit-learn.org/stable/modules/model_persistence.html), since it has "some issues regarding maintainability and security". Better to use json (be careful with numpy array and sklearn objects).

## Training phase

The workflow of the training phase is the following:

![training workflow](https://github.com/MorpheoOrg/morpheo/blob/master/img/training.png?raw=true)

