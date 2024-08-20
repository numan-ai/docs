# Adding Graphs to NNs

In order for a neural network to work with a graph it needs to be able to address nodes in that graph.
For example if we want to add a property to a node we need to know to which node we are adding it.
In regular programs we do it via the node id. Node ids are categorical, meaning that if we have ids 1, 2, 7
all we can say is that they are distinct, distance betwen the numbers give us no information.
In neural networks we can represent categorical values with one-hot encoding, but there is a better way.
