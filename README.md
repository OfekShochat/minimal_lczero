## Minimal LCZero Training

This repo is intended as an easily-understandable, minimal reproduction of the LCZero training setup.

The goal is to eliminate complexity and expose the core net code to allow people to try different approaches
and architectures, without needing to have a deep understanding of RL or the Leela codebase to begin.

### I've never heard of any of this! What is Leela Chess Zero?
Briefly, it's a reimplementation of DeepMind's AlphaGo Zero, applied to chess instead of Go. It's considered
one of the top two chess engines in the world, and usually battles Stockfish for the #1 spot in computer chess
competitions, which is a more classical engine.

### What does the net architecture look like?

Take a look at `tf_net.py` and `tf_layers.py`. Current Leela nets are all ResNets with Squeeze-Excitation blocks,
a reimplementation of which is provided here. This isn't a bad architecture for the problem by any means - 
chess is played on a fixed 8x8 grid, and that lends itself well to convolutional networks.

### Why do you think better architectures are possible, then?

A lot of the fancy tricks used in network training are usually some form of regularization or smuggling inductive bias
in to reduce overfitting. That's because most real-world supervised learning problems are constrained by data. 
Leela is not! Training data is generated by self-play, and vast amounts of it exist. The goal of Leela training
is to *minimize training loss while keeping inference time as low as possible*. The more accurate Leela's predictions,
and the more quickly they can be generated, the better her search and therefore her performance.

This requires quite a different approach from the standard architectures from supervised learning competitions. 
Regularization methods like dropout, which are universal in most supervised computer vision nets, are not helpful in
this regime. Also, the field of computer vision has changed substantially since the AlphaGo Zero paper, with vision
transformers now massively ascendant. There's a huge amount of literature on boosting the performance of transformer
models - can any of that be applied to make a significantly stronger Leela model?

### How can I try my own architecture?

The existing architecture can be seen in `tf_net.py`. Briefly, it consists of an input reshape and convolution, followed
by a stack of residual convolution blocks that make up the bulk of the network, and finally three output heads. These
heads represent Leela's estimates for the **policy** (which move to make), the **value** (how good the current position
is) and the **moves left** (how much longer the game will last after this position). These three outputs
are used to guide Leela's search.

The code here should be simple enough to modify easily if you're familiar with TF/Keras (PyTorch version coming soon!).
Note that the output heads like `ConvolutionalPolicyHead` assume that you're passing them a channels-first tensor,
like you'd get in a convolutional network, with shape (batch, channels, width, height). If you're trying a totally 
different architecture, you may prefer to try the other heads in `tf_layers.py` like `DensePolicyHead` instead. 
These assume the input is a flat tensor of shape (batch, hidden_dim).

### Where do I get data? How do I load it?

The input pipeline is in `tf_data_pipeline.py`. You shouldn't need to modify it, but you can look around if you like!
For smaller nets (e.g. 128 filters, 10 blocks), you may find that you are bottlenecked by the input pipeline instead
of the GPU - if you can find any tricks to speed up the pipeline, please let me know!

As for data, I'm working on building a standard dataset for benchmarking, which I'll probably host as a torrent. It's not 
here yet, unfortunately!

### How do I actually run training?

Just run `tf_train.py`! You'll need a dataset, though - I'm working on that bit.

### Roadmap for this repo

Right now this repo is very unfinished, but the rough plan is:

1) Verify that our training matches the original repo and no bugs have snuck in.
2) Add model checkpointing and Tensorboard logging.
3) Build a standard dataset and validation set and host it somewhere accessible. Also build a smaller dataset that people can experiment with without having to download the whole training set.
4) Add PyTorch versions of everything.