---
title:  "Graph Neural Networks"
search: true
toc: true
categories: 
  - Jekyll
last_modified_at: 2021-01-15T08:06:00-05:00
---

Graph Neural Networks(GNN) are a powerful tool for solving problems on graph-structured inputs.
This article explains the basics of Graph Neural Networks, and their usages in the context of Program Representation and Graph matching.
Graph Neural networks are an extension to traditional neural networks that
take full advantage of the graph structure. 
GNNs are a super exciting sub-area of machine learning that is getting a lot of attention and activity and some impressive 
results recently.
Google team recently used the molecular structure of compounds along with GNNs to predict their aroma
and showed that, that significantly out performed prior methods. GNNs are a broad topic of knowledge. Graphs in general in the context of 
machine learning. [link to the google paper]

### What is a graph? </br>
A graph (G) is a set of nodes/vertices (V) connected by directed/undirected edges (E). We can denote the graph as G (V,E)
Nodes and edges typically come from some expert knowledge or intuition about the problem. Graphs can be;
1. Atoms in molecules
2. Users in a social network 
3. Transportation system or a distribution network, or traffic network
4. Neurons in the brain 
5. A computer program
6. A computer network

With machine learning there are multiple tasks that one can carry out on a graph
1. Node classification: Predict the type of given node. 
   Examples include identifying fake users in a social media platform or any other 
   rating system like Google Maps or Amazon Prime </br>
   ![](/_posts/_gnn/node_classification.png)
2. Link prediction: Predict whether 2 nodes in a network are likely to have a link. 
   Link prediction has uses in friend recommendation, movie recommendation, 
   knowledge graph completion (like associativity between good/bad drugs, and their side effects), 
   metabolic network reconstruction </br>
   ![](/_posts/_gnn/link_prediction.png)
3. Community detection: Identify densely linked clusters of nodes. Community detection has applications in fraud detection
4. Network similarity: How similar are two (sub)networks. This could be used to identifying similar code blocks in a computer programme.

GNN recommendation (PinSage) - Runs in pinterest, Used in Uber eats for food recommendation
Customer x buys product a, customer y buys products a and b
Now the recommender system should recommend product b to the customer x

How to define similarity - in a recommender system
1. Content based: User and item features, the words that describe the user or the item
2. Graph based: User-item interactions, in the form of graph/network structure


Heterogeneous GNN (Decagon) - predic side effects of drugs
Goal-directed generation (GCPN) - molecule generation

Message passing in graph neural networks and node classification

The idea of message parsing in a graph network is a powerful concept
because a lot of the algorithms can be understood from that perspective. 
In simple terms a node in a graph can send and receive messages along its 
edges/connections with its neighbors. This is the essence of label 
propagation algorithms. 

Graph/Node Representation Learning

![](/_posts/_gnn/embed_space.png) | ![](/_posts/_gnn/encoding_graph.png)

In order to use the graphs in neural network setting, we need to embed the graphs in d-dimensional space 
such that similar nodes in the graph are embedded close together, i.e. dissimilar nodes in the graph
should be embedded far apart in the embedding space.
Therefore, an encoding function is needed such that once graph nodes are encoded, the similarity in the
graph space is approximated in the embedding space.

Graph Neural Networks



## References
1. [Course Page - Stanford CS224W: Machine Learning with Graphs](http://web.stanford.edu/class/cs224w/)
1. [YouTube - Stanford CS224W - Machine Learning with Graphs - Fall 2018](https://www.youtube.com/playlist?list=PL-Y8zK4dwCrQyASidb2mjj_itW2-YYx6-) | [Course Material](http://snap.stanford.edu/class/cs224w-2019/)
2. [Git Page - Graph Neural Network Papers](https://github.com/thunlp/GNNPapers)
