# Task 1: Dataset Exploration
-  I first found out the number of edges in the graph along with the types of edges.
- Next, I found out the number of components which make up the graph. I found out that there are 50 components each consisting of either 26 or 27 nodes. 
-  Each such component of 26 or 27 nodes isn't connected to any other nodes. This indicates that each such component is a separate family consisting of 26 or 27 members. 
- These components were visualised separately. The visualisations have been saved as .pngs to the Images directory.
- Metrics such as Modularity, diameter and centrality were calculated seperately for each of the components and visualised through plots.
	- Q=2m1​ij∑​(Aij​−2mki​kj​​)δ(ci​,cj​)
	
	- **Aij​**: The weight of the edge between nodes i and j.
	    
	- **ki​,kj​**: The degrees of the nodes.
	    
	- **m**: The total number of edges.
- Modularity is supposed to indicate how the entire family is branched. A low modularity as seen in this dataset shows that the family is highly connected together. 
- Upon visualisation you can see that some components have a few nodes which doesn't have that many edges with other nodes but the low modularity indicates that the the entire family is closely connected to each other while these few nodes attach themselves to some of these nodes.
- In a graph like this each node in a component can have an edge with every other node in the connected component, we can say that the graph has no incorrect edges but does have quite a few incomplete edges.

# Task 2: Community Detection 

Please Look at  COMMUNITY_DETECTION_REPORT.md

# Task 3: Rule Mining

# Task 4: Link Prediction
# Knowledge Graph Completion using DistMult
---
## 1. Methodology

### 1.1 DistMult: The Chosen Embedding Method

DistMult (_Dist_ributed _Mult_iplicative model) is a knowledge graph embedding method that represents entities and relations as vectors in a continuous vector space. The model learns these embeddings such that valid triples score higher than invalid ones.

#### 1.1.1 Scoring Function

The DistMult scoring function for a triple _(h, r, t)_ is defined as:

```
score(h, r, t) = Σ (h ⊙ r ⊙ t)
```

where _h_, _r_, and _t_ are the embedding vectors for the head entity, relation, and tail entity respectively, ⊙ denotes element-wise multiplication, and Σ represents summation across all dimensions.

#### 1.1.2 Why DistMult?

**Simplicity and Interpretability:** DistMult has one of the simplest scoring functions among KG embedding methods, making it easy to implement, debug, and understand.

**Computational Efficiency:** The bilinear scoring function is computationally efficient, allowing for fast training and inference even on large knowledge graphs.

**Parameter Efficiency:** With only 134,400 trainable parameters for our dataset, DistMult maintains a compact model size while achieving strong performance.
### 1.2 Model Architecture

The DistMult model consists of two embedding layers:

- **Entity Embeddings:** Matrix of size (1,316 × 100) storing vector representations for all entities
- **Relation Embeddings:** Matrix of size (28 × 100) storing vector representations for all relation types

The embedding dimension of 100 was chosen as a balance between model expressiveness and computational efficiency. Both embedding matrices were initialized using Xavier uniform initialization to ensure stable training.

### 1.3 Training Strategy

#### 1.3.1 Negative Sampling

Since the training data contains only positive examples (true relationships), we employ negative sampling to create counter-examples. For each positive triple, we generate 10 negative samples by corrupting the tail entity—replacing it with a randomly selected entity from the knowledge graph.

The negative sampling process includes a filtering mechanism to avoid accidentally creating true triples that exist in the dataset. This ensures that negative samples are genuinely invalid relationships.

#### 1.3.2 Loss Function

We use a margin-based ranking loss with margin γ = 1.0:

```
L = max(0, γ - score(pos) + score(neg))
```

This loss function encourages the model to score positive triples at least γ points higher than negative triples. The margin of 1.0 was selected as a standard choice that provides sufficient separation between positive and negative examples.

### 1.4 Evaluation Methodology

#### 1.4.1 Link Prediction Task

For each test triple _(h, r, t)_, we perform two prediction tasks:

- **Tail Prediction:** Given _(h, r, ?)_, predict the correct tail entity _t_
- **Head Prediction:** Given _(?, r, t)_, predict the correct head entity _h_
- **Relation Prediction:** Given _(h, ?, t)_, predict the correct relation _r_

For each prediction, we score all possible entities and relations and rank them by their scores. The position of the true entity in this ranking determines the prediction quality.

#### 1.4.2 Filtered Ranking

We employ the **filtered ranking** protocol, which is the standard evaluation method in knowledge graph completion. When ranking candidates, we filter out other valid triples that exist in the training or test sets (excluding the target triple itself). This ensures that the model is not penalized for ranking other correct answers highly.

#### 1.4.3 Evaluation Metrics

We report standard link prediction metrics:

**Mean Reciprocal Rank (MRR):** The average of the reciprocal ranks of the correct entity across all predictions.

```
MRR = (1/N) Σ (1/rank_i)
```

**Hits@K:** The proportion of correct entities ranked in the top K positions. We report Hits@1, Hits@3, and Hits@10.

---

## 2. Results and Analysis

### 2.1 Performance Metrics

|**Metric**|**Entity Prediction (H+T)**|**Head Prediction**|**Tail Prediction**|**Relation Prediction**|
|---|---|---|---|---|
|**MRR**|0.6677|0.7151|0.6204|0.4211|
|**Hits@1**|0.4941|0.5525|0.4356|0.0051|
|**Hits@3**|0.8161|0.8678|0.7644|0.8051|
|**Hits@10**|0.9508|0.9729|0.9288|0.9983|
|**Mean Rank**|3.3280|2.4034|4.2525|2.8186|
|**Median Rank**|2.0000|1.0000|2.0000|2.0000|

### 2.2 Performance Analysis

**Exceptional Ranking Performance:** The median rank of 2.0 indicates that for most predictions, the correct entity appears in the top 2 positions. The mean rank of 3.33  and 2.82 is only slightly higher, suggesting that the distribution of ranks is concentrated toward the top positions with minimal outliers.

All metrics are quite amazing except for hits@1 in case of relation prediction and there is a reason to it.

DistMult uses a bilinear diagonal scoring function:

f(h,r,t)=h⊤diag(r)t

Because this formula is mathematically commutative (meaning h⋅r⋅t is the same as t⋅r⋅h), the model cannot distinguish between the **head** and the **tail**.

This means that if the model learns that A is mother of B it mathematically forced to believe that B is mother of A.

However, it performs really well for Hits@3 and near perfectly for Hits@10.
### 2.3 Training Convergence

The training loss decreased from initial values to a final loss of 0.0006 after 100 epochs, demonstrating effective learning. The extremely low final loss suggests the model successfully learned to distinguish between valid and invalid triples in the training set.

The smooth convergence without oscillations indicates that our hyperparameters (learning rate, batch size, margin) were well-chosen for this dataset. The model achieved this performance with 134,400 parameters, demonstrating parameter efficiency relative to the dataset size.

---

## 4. Design Choices and Justification

### 4.1 Hyperparameter Selection

**Embedding Dimension (100):** This dimension provides sufficient capacity to capture relationship patterns without overfitting. With 1,316 entities and 28 relations, a 100-dimensional space offers approximately 3.6 dimensions per relation type, enabling rich representations.

**Negative Sampling Ratio (10:1):** Using 10 negative samples per positive example provides strong signal for learning without excessive computational cost. This ratio ensures the model sees diverse negative examples while keeping training time reasonable.

**Margin (1.0):** A margin of 1.0 enforces reasonable separation between positive and negative examples without being so large that it makes optimization difficult. This is a standard choice that worked well in practice.

**Learning Rate (0.001):** The default Adam learning rate of 0.001 provided stable training. The smooth convergence curve suggests this rate was appropriate for our model and dataset.

**Batch Size (128):** This batch size balances gradient estimate quality with memory efficiency. With 13,821 training triples, we process approximately 108 batches per epoch, providing frequent parameter updates.

**Training Epochs (100):** 100 epochs provided sufficient iterations for convergence. The final loss of 0.0006 indicates the model reached a strong local optimum.

### 4.2 Evaluation Protocol Rationale

The filtered ranking protocol is crucial for fair evaluation. Without filtering, the model would be unfairly penalized for ranking other valid answers (that happen to be in the knowledge graph) above the test answer. By filtering out known valid triples during ranking, we ensure metrics reflect the model's ability to distinguish true relationships from truly invalid ones.

Evaluating both head and tail prediction provides a comprehensive assessment of the model's bidirectional reasoning ability, which is essential for knowledge graph completion.

