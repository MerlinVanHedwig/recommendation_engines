# MODEL CONCEPTUAL FRAMEWORK 
>#### **by Greg Murray**
## INTRODUCTION
The recommender system detailed in this paper is a collaborative filtering, hybrid model, employing components of user-based nearest neighbor and item-based nearest neighbor recommenders, basic network theory, and probability theory, implemented in a MapReduce framework. 

There are two models used for determining recommendations for users, the Pearson model ([USER_MOVIE_NETWORK.py](https://github.com/GregMurray30/recommendation_engines/blob/master/USER_MOVIE_NETWORK.py)), and the Gaussian model ([USER_MOVIE_NETWORK_gaussian.py](https://github.com/GregMurray30/recommendation_engines/blob/master/USER_MOVIE_NETWORK_gaussian.py)). Aside from their edge weightings, both the Pearson  and Gaussian networks are modelled identically and
utilize a combination of Dijkstra's shortest path algorithm and spreading activation to assess the network efficiently and subsequently generate item recommendations.

The user-item network is a weighted, non-directed and acyclic graph<sup>[1](#1)</sup> consisting of two node types, user and item, with node centrality typically in a skewed normal or power-law-like distribution.

<p align="center">
  <img src="https://github.com/GregMurray30/recommendation_engines/blob/master/visualizations/node_dist.png" title="Node Distribution">
 </p>
 
**Figure 1:** *A plot of node centrality distribution for a sample of ratings data with count of node connections on the x axis and density (count of nodes) on the y axis. Note that the count of node connections follows a positively skewed normal distribution in this sample.*

## PEARSON NETWORK MODEL

#### USER NODES
Each user node represents an individual user in this network model. The network's edge weights are
calculated as the Pearson correlation coefficient (hence the name) of the user pair's shared item ratings, as has been demonstrated as the most accurate measurement of similarity between users (Herlocker et al. 1999). For two users, X and Y, then, their sample similarity, *r*, is defined as 
![alt text](https://wikimedia.org/api/rest_v1/media/math/render/svg/bd1ccc2979b0fd1c1aec96e386f686ae874f9ec0).


#### ITEM NODES
The second type of node in this model is the item node which represent individual items. Reciprocating the user nodes, the
item network's edge weights are determined by the two item pair's shared user ratings, but this time using cosine similarity. For items A and B then, similarity is defined
![alt text](https://wikimedia.org/api/rest_v1/media/math/render/svg/1d94e5903f7936d3c131e040ef2c51b473dd071d).

#### CROSS EDGES
Cross edges connecting a user node to a item node indicate the user's rating of that movie
where node **u ∈ G<sub>user</sub>**, and node **v ∈ G<sub>movie</sub>**. In order for a high rating 
to correspond to a small distance, a rating-rank (rating of 5=>1, rating of 4=>2, etc.) is used in for the edge weight in place of the actual rating.

In order to accommodate a belief that an increase in degree separation should correspond to 
a decrease in the similarity regardless of the values of the edge weights, a weighting 
parameter **λ** is added to the distance function, *δ(E)*; compounding the the distance for each node traversal originating from the same node type. Formally, 
  
  > **δ<sub>uv</sub>(E<sub>uv</sub>; λ) = λE<sub>uv</sub>**
 
where **nodeType<sub>u</sub>=nodeType<sub>v</sub>.**

#### RECOMMENDATIONS & MODEL ADVANTAGES
Finally, item recommendations are generated by ranking the user-to-item traversals by shortest distances.

One of the advantages of using a dual node-type network model is that the user's item
ratings, her similarity to other users, and other items similarity to items she rated, are not considered in any arbitrary order, but rather assessed simultaneously[<sup>2</sup>](#2). In addition, each node type's edge distances may be weighted according to one's beliefs about the impact that particular type.

## GAUSSIAN NETWORK MODEL

The Gaussian network model is identical to the Pearson model except for the calculation of the
edge weight distances. Where the edge weights in the Pearson model are calculated with the correlation coefficient and cosine similarity, the Gaussian model's edge weights are the probability that the weighted average magnitudinal
difference between two users, or two items, is greater than some designated threshold parameter **θ**. The weighted value of each item's rating difference for a user pair - not to be confused with the pair's edge weight - is one plus the absolute value of the difference of the two ratings, divided by the standard deviation of all the item's ratings. Mathematically, 

> **ω<sub>ab<sub>x</sub></sub>(r<sub>a<sub>x</sub></sub>, r<sub>b<sub>x</sub></sub>, σ<sub>x</sub>)= (1+|r<sub>a<sub>x</sub></sub>-r<sub>b<sub>x</sub></sub>|)/σ<sub>x</sub>**

where *ω<sub>ab<sub>x</sub></sub>* is the weighted rating difference of user pair *a-b* for item x, *r<sub>a<sub>x</sub></sub>* is user a's rating of item x, and *r<sub>b<sub>x</sub></sub>* is user b's rating of item x. 

<p align="center">
  <img src="https://github.com/GregMurray30/recommendation_engines/blob/master/visualizations/constant_rating3.png" title="Constant Rating Differences">
 </p>

**Figure 2:** *The weighted rating difference values (y axis) plotted against the standard deviation (x axis). Each curve represents a constant value for the rating difference and shows how the weighted rating difference varies with the standard deviation of the item's ratings. Note that the standard deviation has more impact on the weighted rating-difference value when there is consensus opinion (σ is small) compared to when there are mixed reviews (σ is large), and that this effect is more dramatic in the rating difference=0 curve (red) than the rating difference=4 (brown) curve.*
 

The intuition behind weighting each rating difference thus is to lend varying importance to items depending on the degree to which there is a consensus of opinion for that item. For example, looking at figure 1 above, two users with a rating difference equal to 0 (red curve) - similar opinions - on an item with standard deviation equal to 1 - a consensus opinion -  will have a weighted rating-difference value of 1. In comparison, in order for a user pair with a rating difference of 3 on an item (green curve) - divergent opinions - to also have a weighted rating-difference value of 1, the standard deviation must be 4 times higher with σ<sub>a</sub>equal to 4, where essentially no one agrees<sup>[3](#3)</sup>.

Because the range of the weighted rating difference is continuous, the model assumes a Gaussian random variable to model the utility (similarity) of any two nodes. 

<p align="center">
  <img src="https://github.com/GregMurray30/recommendation_engines/blob/master/visualizations/gauss_dists.png" title="Gauss Dists">
 </p>
 
**Figure 3:** *Examples of weighted rating-difference (x axis) Gaussian probability density functions for three different user pairs of varying similarity and variance. User pair 1-2 is least probable of being **dissimilar**, followed by user pair 1-4, then user pair 1-3 with the highest probability of being **dissimilar**. If the threshold paramater θ were set at 1, then each user pair's edge distance would be the area under that pair's bell curve to the right of the pink dashed line (note, it is the probability they are **dissimilar** since a longer edge distance means less similar node pairs).*
 
One (major) shortcoming of the probabilistic approach to edge weights is that since the Gaussian probability density is ![alt text](https://wikimedia.org/api/rest_v1/media/math/render/svg/4abaca87a10ecfa77b5a205056523706fe6c9c3f "Title"), it is undefined for samples with a variance (**σ<sup>2</sup>**) of zero. It can be shown that the cumulative distribution function (CDF) for a Gaussian with zero variance is defined as ![alt text](https://wikimedia.org/api/rest_v1/media/math/render/svg/90400cbbc8895d9f3c9a62d7502ed0f077c6ee3b).
However, because many of the instances with zero variance are clearly more a result of small sample size than two users' unwavering similarity, this CDF is not a practical solution to the zero variance problem (which is really a sample size problem). Instead, when variance is zero, and where the mean difference is less than the threshold parameter **θ**, distance is calculated using a sigmoid function, *δ(n)=e<sup>n</sup>/(1000+e<sup>n</sup>)* [<sup>4</sup>](#4), where **n** is the sample size. In the case where the mean difference is greater than the threshold parameter and the variance is zero, the edge is set equal to infinity, effectively removing the two nodes' connection from the network. Formally, distance in this network is calculated where
  
  >**δ<sub>uv</sub>(N(μ<sub>uv</sub>, σ<sub>uv</sub>); θ)=Pr[N(μ<sub>uv</sub>, σ<sub>uv</sub>)>θ]**, when **σ<sub>uv</sub>>0** and **μ<sub>uv</sub><=θ**;
  
  >**δ<sub>uv</sub>(N(μ<sub>uv</sub>, σ<sub>uv</sub>); θ)=1-e<sup>n<sub>uv</uv></sup>/(1000+e<sup>n</sup>)**, when **σ<sub>uv</sub>=0** and **μ<sub>uv</sub><=θ**, where n is the sample size of **E<sub>uv</sub>**;
  
  >**δ<sub>uv</sub>(N(μ<sub>uv</sub>, σ<sub>uv</sub>); θ)= ∞**, otherwise,

where δ<sub>uv</sub> is the edge distance for node pair u-v.
  
<p align="center">
  <img src="https://github.com/GregMurray30/recommendation_engines/blob/master/visualizations/network_ex.png" title="Network_Example">
 </p>
 
**Figure 4:** *A representation of the Gaussian Network Model. The varying thicknesses of each edge line represent different probabilities of similarity (the figure is a visual representation only, in the model the probabilities determine the distance and no notion of edge "thickness" actually exists). Notice that the item and user networks are not two separate clusters, but rather a mesh of the two node types inextricably linked by their complex network of relational edges.*

## TESTING THE MODELS
In order to test the predictive ability of the two models the "leave-one-out" (LOO) cross validation technique is used. In this way the network can be left virtually unchanged whilst composing the training data sets. 

## *NOTES*

>###### 1
> *Outside of the context of a single node path traversal the graph is non-directed and cyclic. It is only when an individual path is being assessed that the graph becomes more rigid and becomes directed, and acylic in its as no cycles are allowed and a node path may not "double back" on itself.*

>###### 2
>*In this iteration of the scalar model the distances in the user network, item network, and the
 user-item cross network are scaled the same, assigning each network equal impact on the
 final recommendation. These edge weights could easily be scaled to accomodate one's
 beliefs on the importance each network has in determining the right item recommendation.
 For instance if one believed that item similarity was more important than user
 similarity, the edge weights in the item network plane could be scaled down by some
 factor.*
 
 >###### 3
 >*Defined thus, a node pair with a rating-difference of 4 can never have a weighted rating-difference value less than 1.25 since the standard deviation for rating differences can never be greater than 4 ((1+4)/4)=1.25)*

 >###### 4
 >*The constant 1 in the logistic function is replaced with 1000 in order to achieve the desired scaling of the resulting quantity*
 
 ## *BIBLIOGRAPHY*
 >J. L. Herlocker, J. A. Konstan, et al., An Algorithmic Framework for Performing Collaborative Filtering , Proceedings of the 22nd Annual International ACM SIGIR Conference, ACM Press, 1999, pp. 230–237.
