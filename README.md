# Baseline Models for MultiNLI Corpus

This is the code we used to establish baselines for the MultiNLI corpus introduced in [A Broad-Coverage Challenge Corpus for Sentence Understanding through Inference](https://arxiv.org/pdf/1704.05426.pdf).

## Data
The MultiNLI and SNLI corpora are both distributed in JSON lines and tab separated value files. 

- MultiNLI data can be downloaded [here](https://www.nyu.edu/projects/bowman/multinli/multinli_0.9.pdf).
- SNLI data can be downloaded [here](https://www.nyu.edu/projects/bowman/multinli/snli_1.0.zip).

## Models
We present three baseline neural network models. These range from a bare-bones model (CBOW), to an elaborate model (ESIM) which has achieved state-of-the-art performance on the SNLI coprus,

- Continuous Bag of Words (CBOW):  in this model, each sentence is represented as the sum of the embedding representations of its
words. This representation is passed to a deep, 3-layers, MLP. Main code for this model is in `models/cbow.py`
- Bi-directional LSTM: in this model, the average of the states of
a bidirectional LSTM RNN is used as the sentence representation. Main code for this model is in `models/bilstm.py`
- Enhanced Sequential Inference Model (ESIM): this is our implementation of the [Chen et al.'s (2017)](https://arxiv.org/pdf/1609.06038v2.pdf) ESIM, without ensembling with a TreeLSTM. Main code for this model is in `models/esim.py`

We use dropout for regularization in all three models. Dropout is implemented at the embedding layer, and after the final non-linearity, before passing the representations to the softmax classifier. 

## Training Models

### Training schemes

The models can be  trained on three different training sets. Each training "scheme" has its own training script.

- To train a model only on SNLI data, 
	- Use `train_snli.py`. 
	- Accuracy on SNLI's dev-set is used to do early stopping. 

- To trian a model on only MultiNLI or on a mixture of MultiNLI and SNLI data, 
	- Use `train_mnli.py`. 
	- The optional `alpha` flag determines what percentage of SNLI data is used in training. The default value for alpha is 0.0, which means the model will be only trained on MultiNLI data. 
	- If `alpha` is a set to a value greater than 0 (and less than 1), an `alpha` percentage of SNLI training data is randomly sampled at the beginning of each epoch. 
	- When using SNLI training data in this scheme, we set `alpha` = 0.15.
	- Accuracy on MultiNLI's matched dev-set is used to do early stopping.

- To train a model on a single MultiNLI genre, 
	- Use `train_genre.py`. 
	- To use this training scheme, you must call the `genre` flag and set it to a valid training genre (`travel`, `fiction`, `slate`, `telephone`, `government`, or `snli`). 
	- Accuracy on the dev-set for the chosen genre is used to do early stopping. 
	- Additionally, logs created with this training scheme contain evaulation statistics by genre. 
	- You can also train a model on SNLI with training scheme if you want genre specific statistics in your logs. 


### Command line flags

To launch any training scheme, there are a couple of required command-line flags, and an array of optional flags. The code concerning all flags can be found in `parameters.py`.

Required flags,

- `model_type`: there are three model types in this repository, `cbow`, `bilstm`, and `cbow`. You must state which model you want to use.
- `model_name`: this is your experiment name. This name will be used the prefix the log and checkpoint files. 

Optional flags,

- `datapath`: path to your directory with MultiNLI, and SNLI data. Default is set to "../data"
- `ckptpath`: path to your directory where you wish to store checkpoint files. Default is set to "../logs"
- `logpath`: path to your directory where you wish to store log files. Default is set to "../logs"
- `emb_to_load`: path to your directory with GloVe data. Default is set to "../data"
- `learning_rate`: the learning rate you wish to use during training. Defulat value is set to 0.0004
- `keep_rate`: the hyper-parameter for dropout rate. `keep_rate` = 1 - dropout-rate. The default value is set to 0.5. For CBOW and BiLSTM models, we used a keep-rate of 0.9, i.e. a dropout rate of 0.1.
- `seq_length`: the maximum sequence length you wish to use. Default value is set to 50. Sentences shorter than `seq_length` are padded to the right. Sentneced longer than `seq-length` are truncated. 
- `emb_train`: boolean flag that determines if the model updates word emebddings during training. Set to True by default.
- `alpha`: only used during `train_mnli` scheme. Determines what percentage of SNLI training data to use in each epoch of training. Default value set to 0.0 (which maked the model train on MultiNLI only).
- `genre`: only used during `train_genre` scheme. Use this flag to set which single genre you wish to train on.
- `test`: boolean used to test a trained model. Call this flag if you wish to load a trained model and test it on MultiNLI dev-sets* and SNLI test-set. When called, the best checkpoint will be used (see section on checkpoints for more details).

 
*Dev-sets are currently used for testing on MultiNLI while the test-sets have not be released. 

### Other parameters

Remaining parameters like the size of hidden layers and word embeddings, minibatch size can be changed directly in `parameters.py`. The default hidden embedding and word embedding size is set to 300, the minibatch size (`batch_size` in the code) is set to 32.

### Sample commands

To train on SNLI data only, here is a sample command,

`PYTHONPATH=$PYTHONPATH:. python train_snli.py cbow cbow-run1 --keep_rate 0.9 --seq_length 25 --emb_train`

where the `model_type` flag is set to `cbow` and can be swapped for `bilstm` or `esim`, and the model_name flag is set to `cbow-run1` and can be changed to a name of your choice.

Similarly, to train on MultiNLI and SNLI data, using a random 15% sample of SNLI data, here is a sample command,

`PYTHONPATH=$PYTHONPATH:. python train_mnli.py bilstm bilstm-run1 --keep_rate 0.9 --alpha 0.15 --emb_train`

To train on just the `travel` genre in MultiNLI data,

`PYTHONPATH=$PYTHONPATH:. python train_genre.py esim esim-run1 --genre travel --emb_train`

### Testing models

To test a trained model, simply call the `test` flag to the command used for training. The best checkpoint will be loaded and used to evaluate the model's performance on the test-set.

For example,

`PYTHONPATH=$PYTHONPATH:. python train_genre.py esim esim-run1 --genre travel --emb_train --test`

### Checkpoints 

We maintain two checkpoints: the most recent checkpoint and the best checkpoint. Every 500 steps, the most recent checkpoint is updated, and we test to see if the dev-set accuracy has improved by at least 0.4%. If the accuracy has gone up by at least 0.4%, then the best checkpoint is updated.


