# Introduction to Morpheo 

## Main concepts

Morpheo is a for-privacy platform which attracts algorithms and data in order to provide accurate and automatic detection of patterns.  

It is backed by a permissioned blockchain to guarantee traceability of all transactions on the platform. 
Transactions include:
- registration of data / algorithms on the platform   
- training of an algorithm on a set of data  
- computation of a prediction on a given data and using a specific trained algorithm

## Morpheo Architecture

Morpheo architecture is organized as below:

![Morpheo Architecture](img/archi.png)


### Viewer

Entry door for data providers. This is part of the data client described in the white-paper.  

Detailed documentation here (TODO).  

### Analytics and Notebook 

Entry door for algorithms providers. This is part of the data client described in the white-paper.  

Detailed documentation:
- [here](https://github.com/MorpheoOrg/morpheo-analytics) for Analytics
- here for Notebook (TODO) 

### Storage 

Wharehouse of encrypted data and algorithms. 

Detailed documentation [here](https://github.com/MorpheoOrg/storage).

### Compute

Secured containers performing learning and prediction tasks.  

Detailed documentation [here](https://github.com/MorpheoOrg/compute).  

### Orchestrator 

Orchestrator of operations on the platform. This is the part which will be translated to a permissioned blockchain (soon). 

Detailed documentation [here](https://morpheoorg.github.io/morpheo-orchestrator).
