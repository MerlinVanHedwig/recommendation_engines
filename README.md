# recommendation_engine
Movie recommendation engine using Apache Spark
Ratings_sample.txt is "::" separated value file in the format [user_id]::[movie_id]::[rating]::[rating_date]. 


MODEL EXPLANATION
This model utitizes basic network theory and Dijkstra's shortest path algorithm to generate movie recommendations.

Hyperplanar User-Movie Network: The hyperplanar, user-movie network consists of three planar networks with nodes in a power-law-esque distribution. The first plane is the user network, which is a weighted, directed, a-cyclic graph (DAG). The second plane is the movie network, which is also a DAG. The two planes are connected by a third plane, user-movie network which is weighted, non-directed and a-cyclic. in the form of cross-edges, which represent each user's rating of the corresponding movie. 

  The user network edge weights are calculated as the average net difference of their shared movie ratings. The weights are directional in that they may be either negative or positive. However, the node pairs do not show up twice (A-B and B-A) in the graph's edge list as would normally be the case in a directed graph as the order of the pairs is indicated by the sign of its weight. The sum of the edge distances is still to be calculated (using Dijkstra's algorithm, explained later) as the net sum so each edge exists as a vector in this model. The intuition behind negative distance is demonstrated in the following example. 
  
   I.e.: User A rates movie X a 2 while user B rates movie X a 5. In addition, user A rates movie Y a 1 while user C rates movie Y a 5. User B has not rated movie Y and user C has not rated movie X. The edge weights are calculated as the difference in the average net ratings, so edge BA is 3 and edge AC is -4 (G(V,E): B--->A<----C). Since user B and user C both differ largely in taste from user A, they are more similar to each other than to user A and therefore the distance should be small. Using This is accomplished by with signed edge weights where the distance between node B and node C is -1, as opposed to a weight of 7 in a non-directed graph. 
   
   There is one issue with directed edge approach which is that 2nd degree connection distance may be equal to - and therefore impact the movie recommendation identically - a 1st degree connection distance. Clearly, this should not be the case as a 1st degree similarity compares directly and should be weighted more heavily. The simple solution to this problem is assigning a weighting function f(E) to each node. This function is applied whenever an intra-network node is traversed. In this model f(e)_N =1.33E N ∈ P_i, where E is the edge traversing a node vector N, and where node vector N is in plane P_i, and i ∈ (1,2)
  
  The movie network edge weights are the net average difference in their shared user ratings. As in the user network, the movie network is a directed graph so the edge weights are signed. For example, if user A rates movie X a 4 and movie Y a 5, then the edge XY weight is -1. 
  
  The user-movie network consists of cross edges spanning the user and movie planes and connect a user to the movie they have rated, where a rate-ranking is used in place of the actual rating (rating of 5 is a 1, rating of 4 is 2, etc.) for the edge weight in order for a high rating to correspond to a small distance.
  
  Finally, movie recommendations are generated by ranking the user-to-movie traversals of the shortest distances. 
  
  One of the advantages of using a hyperplanar network model is that the user's movie ratings, her similarity to other users, and her rated movies similarities to other movies are not applied in some arbitrary priority but rather considered simulataneously in finding the best movie recommendations. In addition, each plane's distances may be weighted appropriately depending on one's attitudes towards their impact*.

 *In this iteration of the model the distances in the user network, movie network, and the user-movie cross network are scaled the same, assigning each network equal impact on the final recommendation. These edge weights could easily be scaled to accomodate one's beliefs on the importance each network has in determining the right movie recommendation. For instance if one believed that movie similarity was more important than user similarity, the edge weights in the movie network plane could be scaled down by some factor sigma.   

