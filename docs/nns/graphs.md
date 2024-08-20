# Adding Graphs to NNs

In order for a neural network to work with a graph it needs to be able to address nodes in that graph.
For example if we want to add a property to a node we need to know to which node we are adding it.
In regular programs we do it via the node id. Node ids are categorical, meaning that if we have ids 1, 2, 7
all we can say is that they are distinct, distance betwen the numbers give us no information.
In neural networks we can represent categorical values with one-hot encoding, but there is a better way.

Let's say we have a graph with four nodes (A, B, C, D):

<div style="text-align: center"><img src="/assets/nn_to_graph1.png" width="400"></div>

And now we want our neural network to be able to point at one of them.
We can do this by outputing two float numbers from 0 to 1.
Let's say the model gave us numbers 0.45 and 0.65, we can use them to uniquely identify a node in the graph:

<div style="text-align: center"><img src="/assets/nn_to_graph2.png" width="400"></div>

We used a two-dimentional space for the graph, but we can add more axis by adding more floaing numbers to the model output.
And we can also divide the 0-1 interval into more parts. Here we divided it into only two parts.

Having 5 axis and diving each one into 10 parts will give us 10 ** 5 = 100,000 possible node indices.
This approach is better because we want to store concepts in this graph and concepts can be related to each other.
Indexing nodes with vectors allows to use vector Euclidean distance as a measure of concept similarity, meaning
that if two concepts are located in a similar position, they have a similar meaning.