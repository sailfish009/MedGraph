# MedGraph: Structural and Temporal Representation Learning of Electronic Medical Records

## Running MedGraph

MedGraph can be used either as an end-to-end predictive healthcare model to output future medical risks or an unsupervised EMR embedding model to obtain visit and code embeddings.

### System requirements

* [Python 3.6](https://www.python.org)
* [Tensorflow 1.14](https://www.tensorflow.org)
* [NetworkX 2.3](https://networkx.github.io)

### Input data format for MedGraph

MedGraph expects a numpy compressed file (`.npz`) with the following elements in `data` directory:

* Mapping dictionaries for node identifier and index in the VC and VV graphs of training, validation and test sets: `ent2vtx_train`, `ent2vtx_valid` & `ent2vtx_test`
* Adjacency matrix of VC structural relations in the training set: `A_vc`
* Visit attribute vector matrices (i.e. patient demographics and hospital utilization) of training, validation and test sets: `X_visits_train`, `X_visits_val` & `X_visits_test`
* Code attribute vector matrix (i.e. tf-idf vectors of ICD code descriptions or multi-hot ICD ancestors in a higher level): `X_codes`
* VV sequences of training, validation and test sets: `vv_train`, `vv_valid` & `vv_test`, in which we have temporal sequences of visits where each visit event is expressed by "\[visit index, input time, output time, auxiliary task label as one- or multi-hot vector\]". For example, for each patient we can represent the visit sequence as a list of tuples: `[[v1, t1, t2, y1], [v2, t2, t3, y2], [v3, t3, t4, y3], ...]`. 

Have a look at the `utils.py` file for more details.

### Running MedGraph script

```
python train.py dataset --embedding_dim=128 --vc_batch_size=128 --vv_batch_size=32 --K=10 --num_epochs=10 --learning_rate=0.001 --is_gauss=True --distance=w2 --is_time_dis=True
```
* `dataset`: name of the EMR dataset
* `embedding_dim`: visit and code embedding dimension (if we learn Gaussian embeddings, then we learn the mean vectors and diagonal covariance vectors of this embedding dimension)
* `vc_batch_size`: batch size of VC bipartite edges
* `vv_batch_size`: batch size of VV event sequences
* `K`: number of negative VC edges for negative sampling
* `alpha`: hyperparameter for structural loss
* `beta`: hyperparameter for temporal loss
* `gamma`: hyperparameter for auxiliary task loss
* `num_epochs`: number of training epochs
* `learning_rate`: learning rate of the Adam optimizer
* `time_dis`: if specified MedGraph makes predictions at each time step of the visit sequence, or if not specified MedGraph makes predictions at the last time step of the visit sequence

If you want to analyse the dataset behaviour using uncertainty modelling:
* `is_gauss`: if specified MedGraph learns Gaussian embeddings for visits and codes, or if not specified MedGraph produces point vector embeddings
* `distance`: if we represent visits and codes as Gaussians, we can define either `w2` (2-nd Wasserstein distance) or `kl` (symmetric KL divergence) as the distance measure


### MedGraph embeddings

Visit and code embeddings for test EMR are saved in `emb` directory as a dictionary in a numpy file (`.npy`). 

## 2-D visualisation of the code embeddings learned from MedGraph

Medgraph produces 128-dimensional code embeddings for ICD-10-CM codes.
Then, we use t-SNE to project these embeddings into 2 dimensions, for visualisation.
Colour of a code indicates its associated CCS class.
You can find the 2-D plot [here](https://bhagya-hettige.github.io/MedGraph).
When you hover on the plot, you can see the ICD code, its definition and the relevant CCS class.
