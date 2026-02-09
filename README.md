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
# Task 4: Link Pradiction
# Knowledge Graph Completion using DistMult

## Link Prediction on Family Relationship Knowledge Graph

---

## 1. Introduction

Real-world knowledge graphs are inherently incomplete. Not all relationships between entities are explicitly stated, yet many can be inferred from existing patterns. Link prediction, also known as Knowledge Graph Completion, aims to predict these missing edges by learning the underlying structure and patterns in the graph.

In the context of family relationships, if we know that Alice is the parent of Bob, and Bob is the parent of Charlie, we should be able to infer that Alice is the grandparent of Charlie, even if this relationship is not explicitly stated in the knowledge graph.

---
## 2. Methodology

### 2.1 DistMult: The Chosen Embedding Method

DistMult (_Dist_ributed _Mult_iplicative model) is a knowledge graph embedding method that represents entities and relations as vectors in a continuous vector space. The model learns these embeddings such that valid triples score higher than invalid ones.

#### 2.1.1 Scoring Function

The DistMult scoring function for a triple _(h, r, t)_ is defined as:

```
score(h, r, t) = Σ (h ⊙ r ⊙ t)
```

where _h_, _r_, and _t_ are the embedding vectors for the head entity, relation, and tail entity respectively, ⊙ denotes element-wise multiplication, and Σ represents summation across all dimensions.

#### 2.1.2 Why DistMult?

**Simplicity and Interpretability:** DistMult has one of the simplest scoring functions among KG embedding methods, making it easy to implement, debug, and understand.

**Computational Efficiency:** The bilinear scoring function is computationally efficient, allowing for fast training and inference even on large knowledge graphs.

**Strong Performance on Symmetric Relations:** Family relationships often include symmetric patterns (siblings, cousins), which DistMult handles naturally.

**Parameter Efficiency:** With only 134,400 trainable parameters for our dataset, DistMult maintains a compact model size while achieving strong performance.
### 2.2 Model Architecture

The DistMult model consists of two embedding layers:

- **Entity Embeddings:** Matrix of size (1,316 × 100) storing vector representations for all entities
- **Relation Embeddings:** Matrix of size (28 × 100) storing vector representations for all relation types

The embedding dimension of 100 was chosen as a balance between model expressiveness and computational efficiency. Both embedding matrices were initialized using Xavier uniform initialization to ensure stable training.

### 2.3 Training Strategy

#### 2.3.1 Negative Sampling

Since the training data contains only positive examples (true relationships), we employ negative sampling to create counter-examples. For each positive triple, we generate 10 negative samples by corrupting the tail entity—replacing it with a randomly selected entity from the knowledge graph.

The negative sampling process includes a filtering mechanism to avoid accidentally creating true triples that exist in the dataset. This ensures that negative samples are genuinely invalid relationships.

#### 2.3.2 Loss Function

We use a margin-based ranking loss with margin γ = 1.0:

```
L = max(0, γ - score(pos) + score(neg))
```

This loss function encourages the model to score positive triples at least γ points higher than negative triples. The margin of 1.0 was selected as a standard choice that provides sufficient separation between positive and negative examples.

#### 2.3.3 Optimization Configuration

- **Optimizer:** Adam optimizer with learning rate 0.001
- **Batch Size:** 128 triples per batch
- **Training Epochs:** 100 epochs
- **Negative Sample Ratio:** 10 negative samples per positive triple

The Adam optimizer was chosen for its adaptive learning rate properties and robust performance. The learning rate of 0.001 provided stable convergence in our experiments.

### 2.4 Evaluation Methodology

#### 2.4.1 Link Prediction Task

For each test triple _(h, r, t)_, we perform two prediction tasks:

- **Tail Prediction:** Given _(h, r, ?)_, predict the correct tail entity _t_
- **Head Prediction:** Given _(?, r, t)_, predict the correct head entity _h_

For each prediction, we score all possible entities and rank them by their scores. The position of the true entity in this ranking determines the prediction quality.

#### 2.4.2 Filtered Ranking

We employ the **filtered ranking** protocol, which is the standard evaluation method in knowledge graph completion. When ranking candidates, we filter out other valid triples that exist in the training or test sets (excluding the target triple itself). This ensures that the model is not penalized for ranking other correct answers highly.

#### 2.4.3 Evaluation Metrics

We report standard link prediction metrics:

**Mean Reciprocal Rank (MRR):** The average of the reciprocal ranks of the correct entity across all predictions.

```
MRR = (1/N) Σ (1/rank_i)
```

**Hits@K:** The proportion of correct entities ranked in the top K positions. We report Hits@1, Hits@3, and Hits@10.

---

## 3. Results and Analysis

### 3.1 Performance Metrics

The trained DistMult model achieved the following performance on the test set:

|Metric|Score|
|---|---|
|**MRR**|**0.6677**|
|Hits@1|0.4941|
|Hits@3|0.8161|
|Hits@10|0.9508|
|Mean Rank|3.33|
|Median Rank|2.0|

### 3.2 Performance Analysis

**Exceptional Ranking Performance:** The median rank of 2.0 indicates that for most predictions, the correct entity appears in the top 2 positions. The mean rank of 3.33 is only slightly higher, suggesting that the distribution of ranks is concentrated toward the top positions with minimal outliers.

**Strong Top-1 Accuracy:** A Hits@1 score of 0.4941 means that in nearly 50% of predictions, the model identifies the correct entity as its top choice. This demonstrates strong discriminative power for family relationship patterns.

**High Recall in Top-10:** The Hits@10 score of 0.9508 indicates that 95% of correct entities appear within the top 10 predictions. This is particularly valuable for applications where presenting a small set of candidates to users is acceptable.

**Rapid Growth in Hits@K:** The progression from Hits@1 (49.4%) to Hits@3 (81.6%) to Hits@10 (95.1%) shows that most correct answers, if not ranked first, appear within the top few positions. This suggests the model has learned meaningful representations of relationship patterns.

**Effective MRR:** An MRR of 0.6677 is considered strong for knowledge graph completion tasks. This metric accounts for the exact position of correct predictions, and our score indicates that on average, correct entities are ranked very highly.

### 3.3 Training Convergence

The training loss decreased from initial values to a final loss of 0.0006 after 100 epochs, demonstrating effective learning. The extremely low final loss suggests the model successfully learned to distinguish between valid and invalid triples in the training set.

The smooth convergence without oscillations indicates that our hyperparameters (learning rate, batch size, margin) were well-chosen for this dataset. The model achieved this performance with 134,400 parameters, demonstrating parameter efficiency relative to the dataset size.

### 3.4 Key Insights

**Pattern Learning:** The exceptionally low mean rank (3.33) and median rank (2.0) suggest that family relationships exhibit strong structural patterns that embedding methods can effectively capture.

**Practical Utility:** The high Hits@3 score (81.6%) indicates that even when the model doesn't rank the correct answer first, it typically places it within the top three candidates, making the system highly practical for interactive applications.

**Parameter Efficiency:** The model's ability to achieve such strong performance with only 134,400 parameters demonstrates the power of distributed representations. Each entity and relation is represented as a 100-dimensional vector, yet these compact representations encode sufficient information to distinguish between 28 different relationship types across 1,316 entities.

**Model Suitability:** The strong results (MRR of 0.6677) validate the choice of DistMult, demonstrating that a simpler model can achieve excellent performance when well-suited to the data characteristics (symmetric relationships, moderate graph size).

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

---

## 5. Technical Specifications

|Specification|Value|
|---|---|
|Model Type|DistMult|
|Embedding Dimension|100|
|Total Parameters|134,400|
|Loss Function|Margin Ranking (γ=1.0)|
|Optimizer|Adam (lr=0.001)|
|Batch Size|128|
|Training Epochs|100|
|Negative Sampling Ratio|10:1|
|Final Training Loss|0.0006|
|**Test Performance**||
|MRR|0.6677|
|Hits@1|0.4941|
|Hits@3|0.8161|
|Hits@10|0.9508|
|Mean Rank|3.33|
|Median Rank|2.0|

---