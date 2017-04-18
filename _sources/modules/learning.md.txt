# Machine learning mechanisms on Morpheo

## Definition of a machine learning problem on Morpheo

The Morpheo platform can handle different machine learning problems. 
Problems are defined by members of the platform administration.  
The first problem addressed by the platform is sleep stages classification, other potential problems are sleep apnea detection, insomnia detection. 
The definition of a problem requires the creation of a `problem workflow` and a set of data with corresponding targets.  
A `problem workflow` mainly defines what are the **data targets** and the **performance metric** used to evaluate machine learning models.
A new `problem` must be registered in the `Orchestrator` by specifying the UUID of the `problem workflow` and other parameters, such as UUIDs of test data (see in [the Orchestrator documentation for more details on Problem](https://morpheoorg.github.io/morpheo-orchestrator/modules/collections.html#collection-problem)). 

## Training on Morpheo 

### Definition of Training tasks 

For a given problem, different algorithms can be submitted. 
The submission is made through `Analytics` and managed by the `Orchestrator`, which registers the new algorithm and creates associated training tasks. 
We call an `algorithm` a problem solution that has not been trained, and a `model` when this algorithm has been trained.   
**Training tasks are created in two cases**:  
- when an algorithm is submitted.  
- when new data are submitted, models are updated.   
It implies that **only algorithms supporting online learning** can be submitted to the platform.   

 
Training tasks are specified by the `Orchestrator` with the definition of a `learnuplet` (see in [the Orchestrator documentation for more details on Learnuplet](https://morpheoorg.github.io/morpheo-orchestrator/modules/collections.html#collection-learnuplet)), and are consumed by `Compute`.
`Compute` does the training based on the `problem workflow`, saves the resulting `model` in `Storage` after encryption, and sends the performances to the `Orchestrator`.  

Performances are computed on a **test dataset, fixed for a given problem and not accessible**.  
For now, no cross-validation is done and the **performance is public** (no public and private leaderboard). 
This choice is motivated by the fact that the platform is data-oriented: each time new data is uploaded, models are updated, which limit overfitting possibilities. 
This might change latter if needed.

Performances are also computed on each train data and send by `Compute` to the `Orchestrator` as feedback on the training for `Analytics`. 

#### Training hypothesis summary

- model training at algorithm submission and at new data upload 
- submission of algortihms supporting online learning only  
- no cross-validation  
- model performance transparency (no public and private leaderboard)  

### Algorithm submission

An algorithm is submitted to the platform via `Analytics`, which stores it to `Storage` and registers it to `Orchestrator` (see in the [Orchestrator documentation for details about algorithm registration](https://morpheoorg.github.io/morpheo-orchestrator/modules/collections.html#collection-algo)).   
Algorithms submitted to the platform must support **online learning**, since models are updated each time new data are uploaded.  
This is the case for all **neural networks**.  
Algorithms implemented in the package `scikit learn` can be used if they offer a `partial_fit` method (or a `warm_start` option, TODO: check...).

A submission corresponds to the submission of a [Docker](https://docs.docker.com/engine/getstarted/). It must contain the following elements:  
- a `train.sh` 


An example of a submission is given for sleep stages classification [here](https://github.com/MorpheoOrg/hypnogram-wf).

##### Note about saving trained models:  
Saving format is let free for now, since it is part of the code submitted for an algorithm.
- for neural networks, best solution is to save them in h5 files
- for sklearn models, avoid using [Pickle](http://scikit-learn.org/stable/modules/model_persistence.html), since it has "some issues regarding maintainability and security". Better to use json (be careful with numpy array and sklearn objects).

### Training phase

The workflow of the training phase is the following:

![training workflow](img/training.png)

## Predictions on Morpheo 

A user can request a prediction on the platform using the `Viewer`, which transfers the request to the `Orchestrator`. 
To see how to request a prediction to the Orchestrator, see the [Orchestrator documentation](https://morpheoorg.github.io/morpheo-orchestrator/modules/collections.html#collection-preduplet)
