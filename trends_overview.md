# Kaldi ASR: Industry Users -- Summary of the meeting #

* Prepared by Desh Raj, Yiming Wang, Jonathan Chang, Paola Garcia
* October 1, 2020.

## Introduction (Sanjeev Khudanpur) ##

This K2 Townhall session was planned to understand trends in frameworks and hardware acceleration with creators of deep learning frameworks and computing infrastructures, and the interaction with K2. Sanjeev Khudanpur gave a brief introduction to the evolution of the toolkit. ASR is advancing rapidly, but Kaldi has maintained its quality with time. He emphasized that Kaldi is an open-source project and part of its success relies on its more than 300 contributors. Kaldi is preparing for major revisions such as integration to other deep neural network frameworks, automatic differentiation through the FST, and streamlining for data ingestion. The goal of this meeting is to get insights and advice from experts in deep learning frameworks that have experience with speech recognition at scale to make K2 a successful toolkit.

## K2 (Dan Povey) ##

Similar to the previous meeting, Dan Povey explained his vision of the next Kaldi.  The idea is to start from scratch to prepare the new K2 for Pytorch and TensorFlow. There was an effort to add Python wrapping parts, but the design was not very elegant and complex. In the end, this approach was finally abandoned. 

The overall plan is to have different modules: K2 for the sequence modeling (automata on the GPU), Lhotse for the data preparation, and an external toolkit for the neural network (Pytorch or TensorFlow). The recipes’ structure will be decided later, but wouldn’t be part of this repository. They would have their own space.

The basic goal is to add Finite State Acceptor (FSA) as an object compatible with Pytorch and other toolkits as well.  The idea is that FSA supports standard operations: pruned composition, determinization, n-best rescoring which have a fast GPU implementation. FSAs are differentiable, so we can view CTC and LF-MMI as FSA.

He also showed code examples and emphasized the implementation. K2 will be mostly C++/Cuda. He also mentioned Facebook’s GTN (differentiable weighted finite-state transducers) project. The key difference is that K2 will be very efficient on the GPU using data structure similar to “RaggedTensor” which is a list of lists (arbitrarily nesting vector<vector<int>>).   The dependencies include cub and pybind11. The integration with PyTorch and TensorFlow is at the Python level. Autograd can be used since it allows the algorithms to be pure FSAs, and it is implemented at Python level. Confidence estimation based on k2 should be more accurate than those based on lattice probabilities. He gave a final code example on how to perform the confidence estimation based on K2. 

The plan is to release the code on November 1st. The initial release will be CTC and LF-MMI.  The later work will be mostly on the recipes that use K2 and Lhotse. 


## Lhotse (Piotr Zelasko) ##

Piotr gave a Lhotse teaser. Lhotse is Python-centric making it easier for a broader audience. It will have a clean separation between corpus and the task that is being solved. It will have PyTorch Datasets classes for various tasks. Lhotse will use “cuts” and supervision for the data preparation. He described “cuts”  as Lhotse’s workforce and its functionalities. For example, RecordingSet manifest describes audio, SupervisionSet manifest refers to the actual speech segment (metadata), and CutSet manifest is the combination of the previous two. The RecordingSet and SupervisionSet are corpus specific. CutSet is the common representation. The workflow would be to have different corpora, perform the “CutSet” and finally create different task datasets. One of the goals is to work with long recordings. He presented a code example on Switchboard VAD using Lhotse. The release will be by the end of the year. 

### Panel Discussion (moderator: Jan "Yenda" Trmal)

**What is happening in deep learning toolkits that Kaldi developers should be aware of?**

1.  Bring in things that PyTorch is building that you can just leverage on top of. (Joe Spisak)
    1. Do not need to worry about quantization, pipeline parallelism or distributed data-parallel training.
    2. New abstractions like auto batching etc. are added
    3. Spanning tensor like ragged tensor
    4. Should be able to deploy it easily to the device.
2. Good experience with quantization on PyTorch with any platform (Brian Kingsbury)
    1. PyTorch has several backend libraries that can deal with CPU (Joe Spisak)
    2. Theoretical speedups of 4x, but should be able to get 2x speedups easily (Joe Spisak)
3. TensorFlow and google prefer to have an intermediate compiler for the hardware/compiler problem (Michael Riley)
    1. Users can use it at high-level and somebody else works on the compiler CPUs or other devices (Michael Riley)
    2. You are focusing K2 on GPUs and it would not immediately translate to a suitable solution (Michael Riley)
    3. If the K2 FSA in cross-platform is really needed, then all these issues could be explained  (Michael Riley)
        1. You can make it work on a wider set of tools (Michael Riley)
        2. Shouldn’t be too hard to make it work across platforms since we mostly use lambdas. (Dan Povey)
        3. RaggedTensor not supported on some platforms. (Michael Riley)
4. One use case that might become important is federated learning and on-device where high parallelizable or too many concurrencies might not be valid. (Ariya Rastrow)
    1. We can just implement different versions of the algorithms that are optimized for CPUs, but they won’t be as efficient. (Dan Povey)
5. Is it on the roadmap to get an efficient implementation for CPU? (Ariya Rastrow)
    1. Yes, definitely. (Dan Povey)
    2. Already have most of CPU implementation done so it should be doable.
6. Back to the question of making it compatible with other platforms. (Michael Riley)
    1. Most of the algorithms are complex composition of operations, but we are manipulating pointers directly on the CPU/GPU, so we are not representing these operations in units. (Dan Povey)
    2. Would be hard to make it compatible with all platforms and also be efficient.
    3. Kernel versions can also be an issue to consider (Michael Riley)
        1. The FSAs operate on whole batches and they run on the same kernel queue, so it is not that heavy. (Dan Povey)
7.  How does NVIDIA feel about using GPUs for more complex structures than matrices (like ragged arrays)? (Jan Trmal)
    1. GPU is a specialized hardware but we want it to be general purpose. (Hugo Braun)
    2. For complex structures, if it is representable in CUDA, we can leverage full power of GPUs. But the code would need to be designed bottom-up with the special GPU structure in mind. (Hugo Braun)
8. Can ragged array be processed in order? (Kim ?)
    1. It is not really sequential. It is only one long array that can be processed in parallel (look up in row IDs and row splits). You don’t have to group by length.  (Dan Povey)


