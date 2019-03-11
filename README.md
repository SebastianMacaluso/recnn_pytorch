
[![DOI](https://zenodo.org/badge/160135404.svg)](https://zenodo.org/badge/latestdoi/160135404)

RECURSIVE NEURAL NETWORK FOR JET PHYSICS 
PyTorch GPU batch training implementation of G. Louppe, K. Cho, C. Becot and K. Cranmer (arXiv:1702.00748)

Sebastian Macaluso - Dic 2, 2018


=================================================================================================
data  (only test samples are provided)
 
- inputTress: All the raw data with the jet clustering history
- input_batches_pad: Batches of input data for the RecNN


=================================================================================================
RecNN
Working dir with the batch training implementation 

-------------------------------------------------------------------------
search_hyperparams.py: main file that calls train.py and evaluate.py, and runs hyperparameters searches

Parameters to specify before running:
sample_name (this specifies the filenames to be used from data/inputTrees)
multi_scan function arguments

To run:
python search_hyperparams.py --gpu=0 &> experiments/simple_rnn_kt/log_gpu0 &

-------------------------------------------------------------------------
train.py: Call all the other files to load the raw data, create the jet trees, create the batches and train the model. This file calls model/recNet.py, model/data_loader.py, model/preprocess.py and utils.py

Parameters to specify before running:
make_batch=True/False (To create the batched dataset from the raw data, or use a previously generated batched dataset)
pT_order=True/False (To reorganize the trees based on pT)
nyu=True/False (To run over the datasets of arXiv:1702.00748. This datasets are not provided here)
sample_name (if running from search_hyperparams.py, then “sample_name” will be overwritten)
sg (signal label)
bg (background label)

To run as a stand-alone code:
CUDA_VISIBLE_DEVICES=1 python -u train.py --model_dir=experiments/nyu_antikt-kt-delphes_test &> experiments/nyu_antikt-kt-delphes_test/log_nyu_antikt-kt-delphes &

-------------------------------------------------------------------------
evaluate.py: loads the weights that give the best val accuracy and gets the accuracy, tpr, fpr, and ROC auc over the test set.

Parameters to specify before running:
nyu
sample_name (if running from search_hyperparams.py, then “sample_name” will be overwritten)

To run as a stand-alone code:
CUDA_VISIBLE_DEVICES=2 python evaluate.py --model_dir=experiments/nyu_antikt-antikt > experiments/nyu_antikt-antikt/log_eval_nyu_antikt-antikt

-------------------------------------------------------------------------
model/recNet.py: model architecture for batch training and accuracy function.

model/data_loader.py: load the raw data and create the batches:

 - Load the jet events and make the trees.
 - Split the sample into train, cross-validation and test with equal number of sg and bg events. Then shuffle each set.
 - Load the jet trees, reorganize the tree by levels, create a batch of N jets by appending the nodes of each jet to each level and add zero padding so that all the levels have the same size
 - Generator function that loads the batches, shifts numpy arrays to torch tensors and feeds the training/validation pipeline
 
 
model/preprocess.py: rewrite and reorganize the jet contents (e.g. add features for each node such as energy, pT, eta, phi, charge, muon ID, etc) 

-------------------------------------------------------------------------
experiments/template_params.json:  Template file that contains all the architecture parameters and training hyperparameters for a specific run. “search_hyperparams.py” modifies these parameters for each scan

experiments/dir_name: dir with all the hyperparameter scan results (weights, log files, results) for each sample/architecture

-------------------------------------------------------------------------
utils.py: auxiliary functions for training, logging, loading hyperparameters from json file, etc.

-------------------------------------------------------------------------
synthesize_results.py: Aggregate metrics of all experiments and outputs a file located at “experiments/dir_name” with a  list where they are sorted. Currently sorts based on best ROC auc.

To run: python synthesize_results.py
