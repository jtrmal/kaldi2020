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


**What is the next big stuff outside ASR (if ASR is an auxiliary science)? What should developers pay attention to (NLP, dialogue, machine translation)?**

1. We cannot say that ASR is ending. Single-channel ASR in clean environment is good, but in natural human conversation, errors are still large. (Jinyu Li)
    1. We need to deal with continuous speech recognition, diarization, overlapped speech, etc. (Jinyu Li)
    2. We are keeping some of that in mind for the design of Lhotse. (Dan Povey)
2. Jinyu Li:  emphasized that ASR problem itself is not solved (Jinyu Li)
    1. I personally feel we should take a lesson from the signal processing people, especially multi-microphone processing. Some of those algorithms might not be solved using the new trend of End-to-end approaches (Dan Povey)
3. Difference in single vs multi channel during JSALT workshop working on AMI. (Vijay Peddinti)
    1. There are a lot of things happening in the wild that are not represented in the curated datasets that we have. (Vijay Peddinti)
    2. Datasets are well-curated which gives the impression that the problem is solved. (Vijay Peddinti)
    3. WER is not the most important metric. (Jian Wu)
        1. ASR from which we can obtain conclusions, but probably not the real ones for the task.
        2. Task-specific optimization is important.
        3. There are unsolved questions like, how to incorporate world knowledge in speech recognition?
        4. A good example is the Google search, when you type it can guess what you want. It could be possible to do similar things in ASR, even though the system does not get all the words, it can infer what you want.
        5. NLP has made a lot of progress recently (e.g. BERT). Need to use ASR in a smarter way.
    4. Maybe we are expecting too much from a neural network. (Jan Trmal)
        1. Are we expecting too much from the neural network? Or are we just relying on that big data to solve the problem? (Jan Trmal)
        2. Lots of companies just rely on big data, but will it scale to multiple languages or zeta-style data stores. (Jan Trmal)
        3. Besides the big data, perhaps we have to ask how to handle the imbalance of the training data so that we can tackle low resource languages. (Jian Wu)
    5. It would be very helpful to have an interface between ASR and NLU. (Ariya Rastrow)
        1. Can backpropagate to do joint training on both.
        2. Will be important to have a structure between ASR and NLU.
        3. Instead of relying too much on the semiring concept, in this framework, each FSA has a quasi field on each arc. Every arc can have a fake field; so that when we do an operation on these, it will be retained after operations like determinization. (Dan Povey)
    6. Training on the right kind of data is more important than huge data. (Katrin Kirchoff)
        1. On-the-fly data selection instead of on-the-fly augmentation where the criteria is specified by the user.
        2. We are putting a lot more emphasis on confidence estimation in K2. (Dan Povey)

**Q&A**

1. Are you envisioning data formats that let you hold data for lots of utterances or lots of segments together (example: HDF5)? (Brian Kingsbury)
	1. Right now, we just have a prototype, but considering an HDF5 format but looking at other options  (example: Apache RO (used in Hugging Face)
	2. Will come down to supporting multiple backends which the user can choose between depending on their requirement.
	3. personally prefer HDF5 format because of interoperability. (Brian Kingsbury)
	4. HDF5 files can have random access, but sequential is faster. (Brian Kingsbury)
2. Joe Spisak would want to connect with Piotr on the data format. (Joe Spisak)
	1. We would probably need to deal with multiple backends (Dan Povey)
	2. We do similar stuff in PyTorch so glad to chat more. (Joe Spisak)
3. Is PyTorch thinking about data storage for multiple different accesses?
	1. DataLoader in PyTorch. But optimizing I/O from multiple sources may need chunking or other solutions which we are thinking about (Christian Puhrsch)
4. For deep multi-turn dialogues, how can we achieve  good accuracy? Key to enable high-valued goals for customers. Lot of work to be done on long-running conversations. (Arindam Mandal)
5. Kaldi is built on top of an HPC platform, but maybe next-gen should be compatible with, say, cloud-based ML platforms. (Jian Wu)
	1. PyTorch people can help there since we work with all cloud platforms. (Joe Spisak)
6. A lot of Kaldi users are people writing their own apps using ASR. Are these your core users for K2 or is it for speech experts? (Vijay Peddinti)
	1. Want it to be easier to use than Kaldi, and without the steep learning curve. (Dan Povey)
7. Could you give the taxonomy of the kind of problems that would benefit from having automatas as the first-class objects? (Michael Riley)
	1. Example: searching over sequences and comparing their merits. The drawback of E2E is that the alternatives are not compared explicitly. Any kind of problem which has a collection of sequences should benefit from this. (Dan Povey)


**Miscellaneous**

1. Opinions about other open-source packages for ASR? (Michael Rilley)
  1. Espresso has quite a lot of users (Dan Povey).
  2. Toolbox from Nvidia that does speech stuff -- Janus (Yenda)
2. Are you planning to maintain the reproducibility set-up so that people can have an easy launching point? (Vijay Peddinti)
  1. Yes, however, we still need to decide the structure of the recipes (one single repository or separate projects). (Dan Povey)
3. How are you planning to address parallelization?	(Vijay Peddinti)
  1. In the short term, we are going to assume that you are training on a single machine with several GPUs. We will probably use the parallelization mechanisms that the toolkits themselves have. (Dan Povey)
4. Is the ability to train on one recording will also carry over for the decoding? (Thomas Schaaf)
  1. Lhotse will make it easy to deal with long recordings. (Dan Povey)
  2. Lhotse can be used for off-line,  and probably straightforward to reuse it for online (Piotr Zelasco)

