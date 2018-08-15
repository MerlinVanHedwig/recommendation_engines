# recommendation_engine
Movie recommendation engine using Apache Spark
Ratings_sample.txt is "::" separated value file in the format [user_id]::[movie_id]::[rating]::[rating_date]. 


MODEL EXPLANATION
This model utitizes basic network theory and Bellman-Ford shortest path algorithm to generate movie recommendations. This model description serves as the conceptual framework only and does not reflect the model's literal implementation in PySpark. 

Multi-Planar User-Movie Network: The multi-planar, user-movie network consists of two planar networks with nodes in a power-law-like distribution. The first plane is the user network, which is a weighted, directed, a-cyclic graph (DAG). The second planar network - also a DAG - is the movie network. Finally, the two planes are connected by a "field" network, the user-movie network. As the name implies, the user-movie network shares nodes with the user and movie networks. It is a weighted, non-directed and a-cyclic graph in the form of cross-edges, which represent each user's rating of each movie. 

  The user network nodes represent users. The network's edge weights are calculated as the average net difference of their shared movie ratings. The weights are directional in that they may be either negative or positive. However, the node pairs do not show up twice (A-B and B-A) in the graph's edge list, as would normally be the case in a directed graph, as the order of the pairs (direction of the edge) is indicated by the sign of its weight. The intuition behind permitting negative weights is demonstrated in the following example. 
  
   I.e.: User A rates movie X a 2 while user B rates movie X a 5. In addition, user A rates movie Y a 1 while user C rates movie Y a 5. User B has not rated movie Y and user C has not rated movie X. Since the edge weights are calculated as the average difference in the net ratings between users, edge BA is 3 and edge AC is -4 (G(V,E): B--->A<----C). Since user B and user C both differ largely in taste from user A, they are more similar to each other than to user A, and therefore the distance should be small. This issue is resolved by using signed edge weights, where the distance between node B and node C is -1, as opposed to a weight of 7 in a non-directed graph. 
   
   There is one problem that arises with directed edge approach in this context: a 2nd degree connection distance may be equal to, and therefore impact the movie recommendation identically, a 1st degree connection distance. Clearly, the degrees of separation should be reflected in the distance formulation. The simple solution to this problem is assigning a weighting function f(E) to each node. This function is applied whenever an intra-network node is traversed. Formally, f(E<sub>ij</sub>) = λE<sub>ij</sub>, where i, j ∈ (1,2); and P<sub>E<sub>ij</sub></sub> ∈ P<sub>m</sub>; where λ is the weighting parameter, and P<sub>m</sub> is plane m.
  
  The movie network nodes represent movies. The network's edge weights are the average net difference in their shared user ratings. As in the user network, the movie network is a directed graph so the edge weights are signed. For example, if user A rates movie X a 4 and movie Y a 5, then the edge XY weight is -1. 
  
  The user-movie network consists of cross edges spanning the user and movie planes and connect a user to the movie they have rated, where a rate-ranking is used in place of the actual rating (rating of 5 is a 1, rating of 4 is 2, etc.) for the edge weight in order for a high rating to correspond to a small distance.
  
  Finally, movie recommendations are generated by ranking the user-to-movie traversals of the shortest distances. 
  
  One of the advantages of using a multi-planar network model is that the user's movie ratings, her similarity to other users, and her rated movies similarities to other movies are not applied with arbitrary priority. In addition, each plane's edge distances may be weighted according to one's beliefs on the impact that particular network*.

 *In this iteration of the model the distances in the user network, movie network, and the user-movie cross network are scaled the same, assigning each network equal impact on the final recommendation. These edge weights could easily be scaled to accomodate one's beliefs on the importance each network has in determining the right movie recommendation. For instance if one believed that movie similarity was more important than user similarity, the edge weights in the movie network plane could be scaled down by some factor λ.   

