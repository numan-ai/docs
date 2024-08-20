# Experimenting with Neural Networks

Subjectivity is central to all knowledge. We manage to handle a lot of sensory information by simplifying it into basic ideas. Instead of thinking about every photon hitting our eyes, we think about the apple that reflected those photons. This way of simplifying helps us process information effectively. However, it also shows how our understanding is personal and subjective. We constantly adapt our behavioural knowledge to the new incoming situation - we "fine-tune" our brains in real-time.

Now, let's recall that LLMs store their vast knowledge in the model weights, for example the "70b" part in "llama3.1 70b" means how many parameters the model has, essentially acting as an indicator of the knowledge capacity. Both behavioural and factual knowledge is stored in weights. And in order to update weights of the model we need to train it further, which is not something we can do on the fly because of the limitations of the backpropagation algorithm.

In our approach we will move the knowledge from weights to an external memory that is stuctured as a graph.