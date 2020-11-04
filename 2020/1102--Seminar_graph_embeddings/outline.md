## Outline talk "Graph embeddings as building blocks for decentralized learning"

### 2 basic strategies for Federated Learning

1. Centralized model updates => server as the bottleneck
2. de-centralized learning => "local spheres" exchange information


### Challenges for decentralized graph learning

* Vantage point: we need some representation, either in raw or model space, which are suitable for federated / de-centralized learning.
  - ensemble methods like Random Forest (Marburg) are "naturally" distributable -> just compute small bags of trees locally and then merge them together globally
  - graphs as well can be seen as naturally distributable via their subgraphs / components / clusters
    + IF the subgraphs are clearly defined (problem in traditional graph clustering)
* We cannot send raw data over the wire
  - so we need to pre-compute local representations & exchange those
  - but at which level? node - cluster - component - subgraph ?
* We have to consider limited bandwidth (= we need a compressed representation )
  - representations should be dense & compressed
  - => low-dimensional embeddings
* We want to avoid the `gossip` problem => only *some* local models should talk to one another
  - which ones should communicate and can we learn that automatically?
* We want to utilize the learned representations in many different tasks
  - no supervised end-2-end learning, as this would result in many different embeddings
* But first things first -> we need good local graph representations


### Embeddings

* Embeddings are dense, low-dimensional representations of a higher-dimensional input space which carry conceptual meaning (in the case of words: semantic meaning)
  - force the embedder to align a similarity metric (usually cosine) inside the embeding space with a similarity metric in the original space
  - e.g. 'word co-occurrence => cosine similarity'
  - BUT, it could generally be any (pairwise) metric in the original space
* ML on graphs is different than on images or sequence data
  - graphs are generally not regular (5 vs. 5000 friends)
  - graphs don't exhibit spatial locality -> we can permute rows and columns of an adj. matrix, and it's still the same graph (not with images)
* We want to construct graphs from embeddings
  - naive approach: (cosine) similarity-based => kNN-graphs
  - do kNN-graphs do not really fit 
  - construction of kNN-graphs is a whole area in itself, with approaches like `spatial Partitioning`, `random projections` etc.
  - can we also construct similarity graphs from a graph?? -> In principle yes:
    0. overlapping neighborhoods? - OR - 
    1. conduct random walks of length $k$ from each node
    2. group the walks by node label
    3. measure the pairwise Jaccard distance of nodes w.r.t. the nodes on their random walks
    4. of course, this only works if the graph is well-connected (unreachable similar nodes will never be discovered).
* We want to generate embeddings from graphs
  - we do not "only" want to use content (feature vectors of nodes) to yield our embeddings
  - but also the `network structure` should flow into the representation
  - explain GraphSAGE (or better: neighborhood aggregation methods)
    - the idea goes back to Weisfeiler-Lehman Kernels, which assign each node a compressed label based on the initial label of it's neighbours - and it repeats this for all nodes until two labels of $G$ and $G'$ diverge, then it stops - if it never stops, two graphs *might* be isomorphic (but it's not guaranteed)
    - 1st sample a `computation graph` for each and every node in the graph
    - this can be done for each node individually & therefore in parallel, but might produce great overlap in sampled nodes (if stored separately)
    - feature vectors of each node are then propagated *"inwards"* towards the center node of a `compute graph`
    - weight matrices at each layer => those weight matrices are shared => this is actually a miracle - why would this work throughout a graph with possible different substructures / topologies / scale effects...??
      + the "world" looks very different for nodes in the center of a cluster than to those on the periphery with connections to other clusters... 
    - the greater the distance over which we aggregate, the more the node representations will converge (resulting in ONE graph representation in the extreme case)
* Can those two approaches work in tandem & improve upon each other?
  - embeddings can capture similarity between nodes that are not reachable from one another
  - embeddings can capture similarity between nodes that are far from one another in a (general) input graph (like co-authorship or co-purchase)
  - what does the farther-distance structure in a similarity graph represent? Are there transitive effects in "similarity space" and how will they influence embeddings


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
  - quick experiment: show 2 pics and let people guess which one gives the better recommendations


### Approaches / Research avenues

1. Embeddings -> kNN (similarity) recommender -> rule extraction from user feedback -> graphs -> better embeddings
   - can apply to product recommendations or to regions of a histo-pathological image (which cells are the same)
   - could elicit counterfactuals (what are similar points that fall into a different class) -> which criteria lead to the class change?
   - since we specifically want to utilize human feedback, we're 
2. Embeddings -> kNN-graph -> better embeddings -> better graph -> ...?
   - needs kNN-graph construction
   - then fed into GraphSAGE
   - evaluation
3. Embedding spaces representing different metrics (other than similarity) in the input space
   - no experimental setup yet



### Future Challenges

* hierarchical representation of local (sub)graphs -> which works best?
  - aggregation / concatenation (mean/max/etc.)
  - a collection of virtual nodes spanning interesting clusters / walks of nodes
  - sampling strategies such as Random Walks, where the local graph structure itself is translated to fixed-size vectors -> then fed to an embedder
* semantic alignment
  - under the assumption that when exchanging representation, the recipient can actually *decode* their meaning
  - => DIFFERENT SEMANTIC CONCEPT SPACES because of *local geometry*
  - how to align those local geometries efficiently is an open problem
    + pre-train local embeddings as a whole, then try to align them in a post-processing step (has been done via the smoo )
  - => SEE THE WHOLE DECENTRALIZED DATA SET AS A DISTRIBUTED GRAPH
  - since we're passing messages along edges, we can treat a local sphere or parts thereof as a subgraph, compute it's representation, then send them along an imaginary edge to a different local sphere
  - QUESTION: do we connect spheres 1:1 with only one possible edge or should multiple regions / sub(sub-)graphs of one local sphere connect to different regions in another (level of granularity)?
* We want to learn a collaboration graph
  - who talks to whom?
  - learning suitable partners via
* explainability
  - how do we trace relevant walks / paths / regions throughout a distributed graph?
  - especially since we only know embeddings & not distinct nodes / edges in other local spheres...??
* Once we have a *"lego toolkit"* of methods, metrics & a software pipeline, we can aim at more ambitious goals:
   1. extract graphs from histopatho images including labels for nodes / edges
    - cell graph extraction methods already exist
    - show our earlier graph extractor (fasttext as inspiration...)
   2. extract graphs from unstructured text (patient data), medial knowledge bases, *omics etc.
   3. attempt to fuse them into a multimodal graph living in a shared concept space (same embedding dimensions)


### References

* Nino Shervashidze, Pascal Schweitzer, Erik Jan van Leeuwen, Kurt Mehlhorn, and Karsten M. Borgwardt. 2011. Weisfeiler-Lehman Graph Kernels. J. Mach. Learn. Res. 12, null (2/1/2011), 2539–2561.
* Leskovec GraphSage
* Leskovec DiffPool
* Leskovec Group - graph construction video
* Bellet ??
* Sanjoy Dasgupta and Yoav Freund. 2008. Random projection trees and low dimensional manifolds. In Proceedings of the fortieth annual ACM symposium on Theory of computing (STOC '08). Association for Computing Machinery, New York, NY, USA, 537–546. DOI:https://doi.org/10.1145/1374376.1374452
https://global.gotomeeting.com/join/287241213
* Tomas Mikolov, Ilya Sutskever, Kai Chen, Greg Corrado, and Jeffrey Dean. 2013. Distributed representations of words and phrases and their compositionality. In Proceedings of the 26th International Conference on Neural Information Processing Systems - Volume 2 (NIPS'13). Curran Associates Inc., Red Hook, NY, USA, 3111–3119.
* William L. Hamilton, Rex Ying, and Jure Leskovec. 2017. Inductive representation learning on large graphs. In Proceedings of the 31st International Conference on Neural Information Processing Systems (NIPS'17). Curran Associates Inc., Red Hook, NY, USA, 1025–1035

