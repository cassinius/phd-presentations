## Outline talk "Graph embeddings as building blocks for decentralized learning"

### 2 basic strategies for Federated Learning

1. Centralized model updates => server as the bottleneck
2. de-centralized learning => "local spheres" exchange information


### Challenges for decentralized graph learning

* We cannot send raw data over the wire
  - so we need to pre-compute local representations & exchange those
* We have to consider limited bandwidth (= we need a compressed representation )
  - representations should be dense & compressed
  - => low-dimensional embeddings
* We want to avoid the `gossip` problem => only *some* local models should talk to one another
  - which ones and 
* We want to utilize the learned representations in many different tasks
  - no supervised end-2-end learning




### Embeddings

* We want to construct graphs from embeddings
  - naive approach: similarity-based => kNN-graphs
  - do kNN-graphs do not really fit 
  - construction of kNN-graphs is a whole area in itself, with approaches like `spatial Trees`, `random projections` and `kNN descent`
* We want to generate embeddings from graphs
  - we do not "only" want to use content (feature vectors of nodes) to yield our embeddings
  - but also the `network structure` should flow into the representation
  - explain GraphSAGE (or better: neighborhood aggregation methods)
    - 1st sample a `computation graph` for each and every node in the graph
    - this might produce great overlap, but can be done for each node individually & therefore in parallel
    - the idea goes back to Weisfeiler-Lehman Kernels, which assign each node a compressed label based on the initial label of it's neighbours
    - 
* Can those two approaches work in tandem & improve upon each other?
  - embeddings can capture similarity between nodes that are not reachable from one another
  - embeddings can capture similarity between nodes that are far from one another in a (general) input graph (like co-authorship or co-purchase)
  - graph 
* We want to learn a collaboration graph
  - who talks to whom?
  - 


### Infrastructure

* Datasets: for practical reasons (lack of Pathologist availability, especially during Covid-19)
  - two product datasets (Flipkart & Amazon)
  - two skills/occupation datasets (Esco & Onet) as well as a 
  - job recommender dataset by Kaggle (will only come into play later, using the Esco & Onet  )
* Embedding algorithms are w2v & fasttext skipgram
  - trained on 5 million products
  - trained on an integrated Esco / Onet corpus
* Wrote a kNN similarity API
  - takes any arbitrary text, maps it into the embedding space using tokenization (words or n-grams)
  - returns the k nearest neighbors according to a specified target set
  - intended to use the similarity API to construct kNN-Graphs, but this turned out to be a sloppy idea
    + kNN's + max distance is better
    + whole class of algorithms exist for that, still evaluating the approaches...
* using Tensorflow projector for embedding visualization
  - quick experiment: show 2 pics and let people guess which gives the better recommendations



### Challenges

* semantic alignment
  - under the assumption that when exchanging representation, the recipient can actually *decode* their meaning
  - => DIFFERENT SEMANTIC CONCEPT SPACES because of *local geometry*
  - how to align those local geometries efficiently is an open problem
    + pre-train local embeddings as a whole, then try to align them in a pre-processing step (needs whole )
  - => SEE THE WHOLE DECENTRALIZED DATA SET AS A DISTRIBUTED GRAPH
  - since we're passing messages along edges, we can treat a local sphere or parts thereof as a subgraph, compute it's representation, then send them along an imaginary edge to a different local sphere
  - QUESTION: do we connect 
* explainability
  - 
* One we have a *"lego toolkit"* of methods, metrics & a software pipeline, we can aim at more ambitious goals:
   1. extract graphs from histopatho images including labels for nodes / edges
    - cell graph extraction methods already exist
   2. extract graphs from unstructured text (patient data), medial knowledge bases, *omics etc.
   3. attempt to fuse them into a multimodal graph living in a shared concept space (same embedding dimensions)

