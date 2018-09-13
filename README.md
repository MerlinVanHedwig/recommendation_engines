# MODEL CONCEPTUAL FRAMEWORK 
>#### **by Greg Murray**
## INTRODUCTION
The recommender system detailed in this paper is a collaborative filtering, hybrid model which utilizes components of user-based nearest neighbor and item-based nearest neighbor recommenders, basic network theory, and probability theory implemented in a MapReduce framework. 

There are two models used for determining recommendations for users, the Pearson model ([USER_MOVIE_NETWORK.py](https://github.com/GregMurray30/recommendation_engines/blob/master/USER_MOVIE_NETWORK.py)), and the Gaussian model ([USER_MOVIE_NETWORK_gaussian.py](https://github.com/GregMurray30/recommendation_engines/blob/master/USER_MOVIE_NETWORK_gaussian.py)). Aside from their edge weightings, both the Pearson  and Gaussian networks are modelled identically and
utilize a combination of Dijkstra's shortest path algorithm and spreading activation to assess the network efficiently and subsequently generate movie[<sup>1</sup>](#1) recommendations.

The user-item network is a weighted, non-directed<sup>[2](#2)</sup> and acyclic graph consisting of two node types, user and item, with node centrality typically in a skewed normal or power-law-like distribution.

<p align="center">
  <img src="https://github.com/GregMurray30/recommendation_engines/blob/master/visualizations/node_dist.png" title="Node Distribution">
 </p>
 
**Figure 1:** *A plot of node centrality distribution for a sample of ratings data with count of node connections on the x axis and density (count of nodes) on the y axis. Note that the count of node connections follows a positively skewed normal distribution in this sample.*

## PEARSON NETWORK MODEL
#### USER NODES
Each user node represents an individual user in this network model. The network's edge weights are
calculated as the Pearson correlation coefficient (hence the name) of the two users' shared-items' ratings, as has been demonstrated as the most accurate measurement of similarity between users (Herlocker et al. 1999). For two users, X and Y, then, their sample similarity is defined as 
![alt text](https://wikimedia.org/api/rest_v1/media/math/render/svg/bd1ccc2979b0fd1c1aec96e386f686ae874f9ec0).


#### ITEM NODES
The second type of node in this model is the item node which represent individual items. Reciprocating the user nodes, the
item network's edge weights are determined by the two items' shared-users' ratings, but this time using cosine similarity. For items A and B then, similarity is defined
![alt text](https://wikimedia.org/api/rest_v1/media/math/render/svg/1d94e5903f7936d3c131e040ef2c51b473dd071d).

#### CROSS EDGES
Cross edges connecting a user node to a item node indicate the user's rating of that movie
where node **u ∈ G<sub>user</sub>**, and node **v ∈ G<sub>movie</sub>**. In order for a high rating 
to correspond to a small distance, a rating-rank (rating of 5=>1, rating of 4=>2, etc.) is used in for the edge weight in place of the actual rating.

In order to accommodate a belief that an increase in degree separation should correspond to 
a decrease in the similarity regardless of the values of the edge weights, a weighting 
parameter **λ** is added to the distance function, *δ(E)*; compounding the the distance for each node traversal originating from the same node type. Formally, 
  
  > **δ(E<sub>uv</sub>; λ) = λE<sub>uv</sub>**
 
where **nodeType<sub>u</sub>=nodeType<sub>v</sub>.**

#### RECOMMENDATIONS & MODEL ADVANTAGES
Finally, item recommendations are generated by ranking the user-to-item traversals by shortest distances.

One of the advantages of using a dual-node-type network model is that the user's movie
ratings, her similarity to other users, and her rated items' similarities to other
items are not considered in any arbitrary order, but rather assessed simultaneously[<sup>2</sup>](#2). 
In addition, each node type's edge distances may be weighted according to one's beliefs about the impact 
that particular type.

## GAUSSIAN NETWORK MODEL:

The Gaussian network model is identical to the Pearson network except for the calculation of the
edge weight distances. Where the edge weights in the Pearson model are calculated with the correlation coefficient and cosine similarity, the Gaussian model's edge weights are the probability that the weighted average magnitudinal
difference between two users, or two items, is greater than some designated threshold parameter **θ**. The weighted value of each item's rating difference for a user pair - not to be confused with the node pair's edge weight which is determined by the distribution of *all* the weighted rating difference values for two users - is the absolute value of the difference of the two ratings divided by the standard deviation of the item's ratings. Mathematically, 

> **w<sub>u<sub>a</sub></sub>(r<sub>u<sub>a</sub></sub>, r<sub>v<sub>a</sub></sub>, σ<sub>a</sub>)= |(r<sub>u<sub>a</sub></sub>-r<sub>v<sub>a</sub></sub>)|/σ<sub>a</sub>**,

where r<sub>u<sub>a</sub></sub> is the user u rating of item a and r<sub>v<sub>a</sub></sub> is the user v rating of item a. The intuition behind weighting each rating difference thus is to lend varying importance to items depending on the degree to which there is a consensus of opinion for that item. For example, two users with a divergent opinions (ie: rating difference of 4) on an item with consensus opinion (low variance) will receive a relatively high weighted rating difference. In contrast, two users with very similar opinions (ie: rating difference of 1) on an item with "mixed reviews" (high variance) will have a relatively low weighted rating difference.

<p align="center">
  <img src="https://github.com/GregMurray30/recommendation_engines/blob/master/visualizations/constant_var.png" title="Constant Variance">
   <img src="https://github.com/GregMurray30/recommendation_engines/blob/master/visualizations/constant_rating.png" title="Constant Rating">
 </p>
 

The model makes the assumption that the utility (similarity) of any two nodes' can be modelled with a Gaussian random variable. 

One major disadvantage of the probabilistic approach to edge weights is that since the Gaussian probability density is ![alt text](https://wikimedia.org/api/rest_v1/media/math/render/svg/4abaca87a10ecfa77b5a205056523706fe6c9c3f "Title"), it is undefined for samples with a variance (**σ<sup>3</sup>**) of zero. It can be shown that the cumulative distribution function (CDF) for a Gaussian with zero variance is defined as ![alt text](https://wikimedia.org/api/rest_v1/media/math/render/svg/90400cbbc8895d9f3c9a62d7502ed0f077c6ee3b).
However, because many of the instances with zero variance are clearly more a result of small sample size than two users' unwavering similarity, this CDF is not a practical solution to the zero variance problem (which is really a sample size problem). Instead, when variance is zero, and where the mean difference is less than the threshold parameter **θ**, distance is calculated using a sigmoid function, *δ(n)=e<sup>n</sup>/(1000+e<sup>n</sup>)* [<sup>4</sup>](#4), where **n** is the sample size. In the case where the mean difference is greater than the threshold parameter and the variance is zero, the edge is set equal to infinity, effectively removing the two nodes' connection from the network. Formally, distance in this network is calculated where
  
  >**δ(E<sub>uv</sub>; θ)=Pr[N(μ<sub>uv</sub>, σ<sub>uv</sub>)>θ]**, when **σ<sub>uv</sub>>0** and **μ<sub>uv</sub><=θ**;
  
  >**δ(E<sub>uv</sub>; θ)=1-e<sup>n<sub>uv</uv></sup>/(1000+e<sup>n</sup>)**, when **σ<sub>uv</sub>=0** and **μ<sub>uv</sub><=θ**, where n is the sample size of **E<sub>uv</sub>**;
  
  >**δ(E<sub>uv</sub>; θ)= ∞**, otherwise
<p align="center">
  <img src="https://github.com/GregMurray30/recommendation_engines/blob/master/visualizations/network_ex.png" title="Network_Example">
 </p>
 
**Figure 2:** *A representation of the Gaussian Network Model. The varying thicknesses of each edge line represent different probabilities of similarity (the figure is a visual representation only, in the model the probabilities determine the distance and no notion of edge "thickness" actually exists). Notice that the item and user networks are not two separate clusters, but rather a mesh of the two node types inextricably linked by their complex network of relational edges.*

## TESTING THE MODELS
In order to test the predictive ability of the two models I utilized the "leave-one-out" (LOO) cross validation technique. In this way the network can be left virtually unchanged whilst composing the training data sets. 

## *NOTES*
>###### 1
>*While items are the recommendation object of interest in these examples, the model is generalizable to any data set that fits a user-product-rating paradigm.*

>###### 2
>* Although the weights are non-directed, the user node pairs still show up twice (*X-Y* and *Y-X*) 
in the graph's edge list where **w<sub>*XY*</sub>=w<sub>*YX*</sub>** during implementation.

 
>###### 3
>*In this iteration of the scalar model the distances in the user network, item network, and the
 user-item cross network are scaled the same, assigning each network equal impact on the
 final recommendation. These edge weights could easily be scaled to accomodate one's
 beliefs on the importance each network has in determining the right item recommendation.
 For instance if one believed that item similarity was more important than user
 similarity, the edge weights in the item network plane could be scaled down by some
 factor.*
 
 >###### 4
 >*The constant 1 in the logistic function is replaced with 1000 in order to achieve the desired scaling of the resulting quantity*

 
 ## *BIBLIOGRAPHY*
 >*J. L. Herlocker, J. A. Konstan, et al., An Algorithmic Framework for Performing Collaborative Filtering , Proceedings of the 22nd Annual International ACM SIGIR Conference, ACM Press, 1999, pp. 230–237.
