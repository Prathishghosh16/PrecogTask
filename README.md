Task 1: Dataset Exploration
-  I first found out the number of edges in the graph along with the types of edges.
- Next, I found out the number of components which make up the graph. I found out that there are 50 components each consisting of either 26 or 27 nodes. 
-  Each such component of 26 or 27 nodes isn't connected to any other nodes. This indicates that each such component is a separate family consisting of 26 or 27 members. 
- These components were visualised separately. The visualisations have been saved as .pngs to the Images directory.
- Metrics such as Modularity, diameter and centrality were calculated seperately for each of the components and visualised through plots.![[Screenshot 2026-02-09 at 10.49.57 PM.png]]
- Modularity is supposed to indicate how the entire family is branched. A low modularity as seen in this dataset shows that the family is highly connected together. 
- Upon visualisation you can see that some components have a few nodes which doesn't have that many edges with other nodes but the low modularity indicates that the the entire family is closely connected to each other while these few nodes attach themselves to some of these nodes.
- In a graph like this each node in a component can have an edge with every other node in the 