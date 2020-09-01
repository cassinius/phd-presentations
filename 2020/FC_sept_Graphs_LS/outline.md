## FC Meeting, Graph talk, outline

> Stay as generic as possible, specific algorithmic details aren't interesting for anyone...


### Progress to date

* original thought was to tackle graphs in a traditional way
  - assume a global (connected) graph
  - distribute it optimally or not via partitioning (KL, MCMF, RW, PR)
* this idea came form an experimental mindset, where we assumed having a certain dataset which we could compute globally -> then split the data & compare with distributed results
* reality however will actually determine datasets and their locality (distribution) for us. Also, graph types will never be text-book pure
  - also, in modern network representation learning approaches (RW-based  / GCNs) strict graph classes are rarely mentioned
* the whole are of Graph Representation Learning (GRL) has become a lot more important to us, since it could help to produce 


### What have we learned

> mostly from literature research - don't remember all the algorithms mentioned...

* cannot use spectral graph theory (e.g. feeding an Adjacency or Laplace matrix into a CNN)
  - this would assume we know all nodes in beforehand
  - every change in the graph would trigger a complete re-computation
  - also unrealistic on distributed graphs, or if parts of the data are sensitive
* cannot use transductive methods (which propagate labels based on some proximity assumption)
* with a random walk (like deep-walk), it depends if the local graphs are sizeable enough for expressive vectors. 
* therefore, neighborhood-aggregation is much more promising
  - can be sampled for each node => built-in parallelization
  - don't need to compute the whole graph at once
  - can learn weight matrices forming an inductive model, so we can classify previously unseen nodes later on
* in the contemporary DL era, we are usually using concept vectors to express things:
  - word embeddings (encoding meaning)
  - structural embeddings (encoding the 'role' of a node in a network)
  - weights from feature extraction layers (depending on whatever the task is)
* VISION: it would make sense to bring features from different modalities (images, text, knowledge graph, time-series, *omics..) into one embedding space, so we can
  - train algorithms to classify/predict on such enriched data bases
  - make explanations more contextual, for the medical experts
  - produce human-understandable counterfactuals (e.g. *"if blood pressure was higher, this cell-cluster would mean something different..."*)


### Next important steps

* Medical university Graz has a dataset of 1.3 million patients with diagnosis spanning multiple organs
  - the data-set is "human generated" (even "doctor" generated)
  - a decition tree has been constructed BY HUMANS (= ground truth)
* interesting questions include
  - can we learn embeddings / node representations directly on that corpus?
  - how do the embeddings / reps change if we learn on a graph initially structured according to human bias (edges between patients in same age cohorts etc...)
  - can we match nodes representing same diagnoses that are expressed in different words? (FUSION)
  - classics like disease classification / clustering
* building a controlled (graph) `"vocabulary"` for interesting regions in images (cell clusters) and their relations
  - e.g. shape descriptions of lymph nodes..
  - which is going to be a Masters thesis
  - we could later use this to extract graphs out of images (like in our age-old demo project) and / or fuse them with other modalities - OR - alternatively learn a joint multi-modal embedding space
* apply the neighborhood aggregation paradigm on distributed graphs
  - can be toys, we just need to compare accuracy with a globally computed result
  - how do we sample neighborhoods across local subgraphs => we DONT, at least not the raw data...
  - can we replace raw data with 
    + a) anonymized feature vectors? 
    + b) already aggregated, representative vectors from relevant other subgraphs (then what is relevant?)
  - how do we ensure that local models generalize well?
    + on the other hand, they should be more accurate (overfit?) on local data => GRAPH ENSEMBLES ??
    + maybe same problem in a big, local graph... because it's parts can have different substructures...
* learn a collaboration graph (see below)
  - this is what we called the `connection surfaces` before...


### Misc (Bellet)

> local training datasets, common feature space $X$ & label space $Y$

* Learn "who to communicate with" by inferring a `graph of similarities` between local spheres
* Collaboratively learn `personalized models` over this graph
* Optimize the models and the graph `jointly, in an alternating fashion`

* Asynchronous time model (parallel)
* Restrict communication to most similar instances
  - similarity via a) `learning task` (objective) or `structural / feature similarity` ?
  - we learn this via a `collaboration task` where the edge weights correspond similarity of learning tasks between instance $i$ and instance $j$