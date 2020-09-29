# Kaldi ASR: Industry Users -- Summary of the meeting #

* Prepared by Desh Raj, Yiming Wang, Jonathan Chang, Paola Garcia
* September 24, 2020.

## Introduction (Sanjeev Khudanpur) ##

The k2 Townhall meeting was planned to discuss the future of Kaldi. Sanjeev Khudanpur gave a brief introduction to the evolution of the toolkit. He emphasized that Kaldi is an open source project and remains one of the leading tools for researchers and small companies. Part of its success relies on its more than 300 contributors. Kaldi started to show its age and it is preparing for major revisions such as integration to other deep neural network frameworks, automatic differentiation through the FST, and streamlining for data ingestion. The goal of these meetings is essentially to understand the what, how, and who. We will need the help of the community.    

## K2 (Dan Povey) ##

Similar to the previous meeting, Dan Povey explained his vision of the next Kaldi.  The idea, now, is to start from scratch to prepare the new K2 for Pytorch and TensorFlow. There was an effort to add Python wrapping parts, but the design was not very elegant and was finally abandoned. 

The overall plan is to have different modules: K2 for the sequence modeling, Lhotse for the data preparation, and an external toolkit for the neural network (Pytorch or TensorFlow). The recipes’ structure will be decided later.

The basic goal is to add Finite State Acceptor (FSA) as an object compatible with Pytorch and other toolkits as well.  The idea is that FSA supports standard operations: pruned composition, determinization, n-best rescoring which have a fast GPU implementation. FSAs are differentiable, so we can view CTC and LF-MMI as FSA.

He also showed code examples and emphasized the implementation. K2 will be mostly C++/Cuda. The key structure in K2 will be similar to `RaggedTensor` which is a list of lists (arbitrarily nesting `vector<vector<int>>`).  Autograd can be used since it allows the algorithms to be pure FSAs, and it is implemented at Python level. Confidence estimation based on k2 should be more accurate than those based on lattice probabilities.
 
We plan to release the code Mid-to-late October. The initial release will be CTC and LF-MMI.

## Lhotse (Piotr Zelasko) ##

Piotr gave a Lhotse teaser. Lhotse is Python-centric making it easier for a broader audience. 
He described “cuts” and its functionalities by using data manifests. For example: RecordingSet describes audio, SupervisionSet refers to the actual speech segment, (metadata) , and CutSet is the combination of the previous two. The RecordingSet and SupervisionSet are corpus specific. CutSet is the common representation.
 
### Panel Discussion (moderator: Mei-Yuh Hwang)

**Inference and runtime**



1. For industry accuracy and run-time performance matters the most (Mike Lei)
    1. The streaming ASR tasks need to have low latency,  and high accuracy
    2.  Consider that industry has constraints, either on the server side on the device side, e.g.  CPU memory consumption requirements
    3. Traditionally Kaldi has been very efficient, K2  should support MKL for server-side,  and continue having C++ code.
    4. Server-side APIs for embedded computing will be needed.
    5. Consider going beyond ASR, including components such as VAD, endpoint detection, confidence estimation, speaker ID etc.
    6. Support for ARM
2. ONNX is not compatible with Transformer-style architectures (Nagendra Goel)
    1. Torchvision not fully functional for all architectures
    2. Would be good idea for someone to make sure all these architectures are supported on these intermediate platforms (C++ style)
3. If K2 decoder natively supports GPU based decoding, it would be of huge advantage. It is a bit tricky to make it work right now (Tao Ma)
4. It would be great if there is C++ reference available for online decoding plus feature extraction implementation (Tao Ma)
5. Kaldi 2018 decoder is very fast in GPUs (Ian Lane)

_<span style="text-decoration:underline;">Observations/commitment</span>_  (Dan Povey):



1. Pytorch and TensorFlow integration should make it easier for things such as multi-core, different architectures. It will require some coding.
2. K2 decoders wouldn’t be as optimized for CPU as current c++ Kaldi decoders because of the GPU-specific data structures, but it will be easier to parallelize to multicore.
3. C++ reference available for online decoding, feature extraction implementation might have to wait for a few months.


#### _<span style="text-decoration:underline;">Inference and Runtime Q&A</span>_



1. Are you considering supporting external LM bias?(Tao Ma)
    1. Writing a wrapper to do LM bias wouldn’t be hard  (Dan Povey)
        1. Our plan is to rely mostly on context-independent phones, we can rely on the neural network to take care of the context.
2. Can k2 decoder leverage GPU? (Tao Ma)
    * K2 can leverage GPU.  Same code will be running in train and inference (Dan Povey)
3. Are you considering K2 decoder to work on CPU as well as regular decoder (Marc Ferras)
    * Decoder will work on CPU, but it would be slower. (Dan Povey)
4. Is there going to be an easy interface between Kaldi decoders and K2 models? (Marc Ferras)
    * Definitely plan to make Kaldi decoders interface well, but not within K2
5. Will there be C++ level feature extraction etc. as well? (Tao Ma)
    * K2 won’t have feature extraction, it will probably be done in Lhotse. Dan hasn't given much thought into how to integrate everything in C++. (Dan Povey)



---
 

**Migrating from Kaldi to K2**



1.  Risk vs Reward. Everyone has their own system based on Kaldi. Swapping over to K2 is a non-trivial investment. (Yishay Carmiel)
    1. 10x factor, i.e., it will be a non-trivial investment to change the system to K2
    2. How to ensure this investment gives positive reward?
    3. Kaldi has already proven itself, not the same for K2. 
    4. For K2, new companies may need to know about use cases, so that they feel comfortable.
    5. Kaldi works for English, perhaps exploring new languages could show the value of K2
2. If components from K2 can be easily pulled into the Kaldi codebase, incremental switching may be possible until complete transition is possible. It would make easier for early adopters of K2 (Nagendra Goel)
3.  If there is a forward-pass library that can be integrated into Kaldi (like Onnx) then K2 models can be used with Kaldi decoders (Karel Vesely)
4. It is a huge change, and industry have to consider benefits vs risks  (Xavier Anguera)
5. Estimating confidences will be great (Raphael Cohen)
6. A guideline for making the transition from Kaldi to K2 will be helpful (Jungsuk)
7. Port the current investments in Kaldi forward to K2 (example: work like Yiming wang did to convert models to ONNX) (Sanjeev Khudanpur)
8. Desire to migrate to k2 but it is going to be difficult from day one. (Nagendra Goel) 
    1. Industry has to think about product requirements.
    2. Take components from k2 and use them in Kaldi.
    3. The key point  is “How do I make a transition plan?”
9. When we start working on recipes, it would be a good idea to post “official” migration guidelines (Guoguo Chen)
    1. People can follow up and make modifications for their needs.
    2. If k2 ends up being way better then it is a ‘no-brainer’ to make the switch.
10. Dan should focus on the engineering of K2 and some other contributor could work on migration (Mike Clark, Sanjeev Khudanpur)
    * The community should allocate some time to contribute (Mei-Yuh Hwang)

_<span style="text-decoration:underline;">Observations/commitment</span>_  (Dan Povey):



1. Graphemic systems should be very easy to implement in K2
2. Rather take bits of Kaldi and use it in K2, over making Kaldi a huge dependency.
3. We will keep in mind how to make incremental switching possible. 
4. K2 will be a single unified system in which you can train your RNN LM and run the inference efficiently (PyTorch, for example).
5. Think about how to transition from Kaldi to K2 (guideline)
6. If needed it would be possible to import data and language directories from Kaldi to K2. However, that is not the direction we want to pursue; the idea is to build a totally new type of system. \


_<span style="text-decoration:underline;">Migrating from Kaldi to K2 (Q&A)</span>_



1. Is there a plan for transition time? (Xavier Anguera)
    * We are maintaining Kaldi, but not implementing new features (Dan Povey)
2. Would K2 be as accurate as Kaldi? (Tao Ma)
    *  It would hopefully be the same by the end of the year. It would be better later because of easier implementation of things like large scale pretraining and RNN LM rescoring. (Dan Povey)
3. Any suggested way to move from Kaldi to K2? (Jungsuk Kim)
    * K2 is more general-purpose; hence, there is no reason why regular Kaldi decoders cannot be cast as K2 FSAs (Dan Povey)
4. So start with K2 decoder and use Kaldi training? (Jungsuk Kim)
    * It is not the first use case we want to address.  We want to start with LFMMI/CTC implementation in PyTorch with simple recipes.
5. Is there going to be model compatibility between Kaldi and K2? (Mei-Yuh Hwang)
    * Probably won’t be compatible, but that is to start with a clean slate and the goal is to be compatible with PyTorch and TensorFlow. We will revisit Kaldi and make K2 better (Dan)
6. Is it possible to have a tool to convert from Kaldi to K2? (Mei-Yuh Hwang)
    * It would be possible, but for the moment we want to focus on FSAs.



---


**Trainability/stability**



1. Training with large data is a problem with current Kaldi (Miguel Jette)
    * Adapting on the fly is also a problem.
2. Training staff to be educated on how to use these tools is a common concern (Nagendra Goel)
    1. Concern shared with academia
    2. Have a bunch of Jupyter notebooks in a separate repository to explore concepts related to ASR
    3. Needs a strong background with FSA (Dan Povey)
    4. Yishay Carmiel would be interested to contribute (Yishay Carmiel)
3. We need such notebooks to educate new people (Sanjeev Khudanpur)
4. Training on the cloud (Wonkyum Lee)
    1. Current Kaldi trains on HPC
    2. Inference happens in the cloud and also the data is in the cloud. 
    3. Would be great if cloud based training can be supported
5. Consider having a list of people working on different separate projects (Jungsuk Kim)
6.  We might need somebody to be in charge of the recipe repo. (Dan Povey)

_<span style="text-decoration:underline;">Observations/commitment</span>_  (Dan Povey):



1. Cloud based training will be a lot easier. K2 is a FSA library and doesn’t deal with I/O.
    1. Lhotse will deal with I/O and can be integrated with cloud computing
    2. Actual parallelization will have to be in the recipe
2. Get nice examples that can work on different platforms (cloud computing).
3. We don’t have a list of people working on different projects; but it will be a good idea to have it eventually.
4. The recipes probably will be Python scripts with shell scripts to set it up.

_<span style="text-decoration:underline;">Trainability/stability </span>_Q/A



1. Will other grid engine platforms be supported (AWS, Google cloud)? (Jungsuk Kim)
    * Kirill Katsnelson had a project to do kaldi on Google Cloud (BurMill). Once we get to training stage, it is possible to modify this project to train K2 models in Google Cloud (Yenda)



---


**Pretrained models**



1. Contribution of pre-trained models (Mohamed El-Geish) 
    1. Having a central hub with models, like a model zoo (borrowing a leaf from NLP/HuggingFace)
    2. Community can contribute their pretrained models e.g. for very specific applications
    3. Upload pretrained models and information about data and how things were trained. 
    4. Also need to guide people on how to use these pretrained models.
    5. How to do inference without installing Kaldi, can be doable. 
    6. Would be good if someone can take ownership of this (Dan Povey)
2. OpenSLR for pre-trained models was a good idea, we have to think further (Sanjeev Khudanpur)
3. Having a pre-trained model makes the task a lot easier e.g. ImageNet (Guoguo Chen) 
    * Would be good to attract non-speech people
4. Especially for non-English languages, would be great to have pretrained models (Raphael Cohen)
5. In NLP, it is impressive that you can build multilingual models for all languages (Mei-Yuh Hwang)

_<span style="text-decoration:underline;">Observations/commitment</span>_  (Dan Povey):



1. K2 will consider having pre-trained models.

_<span style="text-decoration:underline;">Pretrained models (Q&A)</span>_



1. What about better interface between ASR output to NLU tasks? (Themos Stafylakis)
    * should be possible to do all those in K2, but it might be tricky because of normalizations.(Dan Povey)



---


**Audience and How do we market K2?**



1. Following ESPNet example that evolved from ASR to TTS, ST etc, we should aim for K2 to do something similar (Mohamed El-Geish)
    1. Have a framework with K2 which can do several things.
    2. Lhotse is designed with that general-purpose in mind for this (Dan Povey)
2. Kaldi was at first cater towards speech experts. Then it was general machine learning researchers and now it is general engineers that just want a black box. (Mohamed El-Geish)
    1. Not everything has to be  in K2, but should be flexible to adapt them to applications, and have our own adaptors.
    2. Hugging Face example, in which you can add layers on top of layers until you have a high abstraction.
3. ASR is a commodity; need to focus on downstream task (Yishay Carmiel, Mohamed El-Geish)
4. Kaldi has an established brand with a great image of accuracy and industry friendly. It has some limitations (neural network flexibility and its growing complexity) and people expect that the new K2 will fix these issues. (Tao Ma)
    *  K2 will be much more generic. But this is not for marketing since we are providing it at zero-cost anyway. (Dan Povey)

_<span style="text-decoration:underline;">Observations/commitment</span>_  (Dan Povey):



1. K2 is going to be highly accurate but the design is going to be different from Kaldi. 

_<span style="text-decoration:underline;">Audience and How do we market K2? Q&A</span>_



1. How would K2 be better than ESPNet/Espresso/Speech Brain? There needs to be some driving point for people to adopt Kaldi. (Marc Ferras)
    * No objections to espnet. The front end could be espnet and the backend is k2. Doesn’t really see espnet as a competitor But these things should grow organically. (Dan Povey)



---


**End-to-end ASR in K2**



1. Doesn’t like the terminology “e2e” because it’s not well defined. Enlarges the space where different kinds of decoders can be used. K2 will be able to do things that people call e2e. (Dan Povey)
2. Having a separation of AM and LM is easy for industrial users. (Tao Ma)
    * It will be possible in K2 (Dan Povey)
1. If Google releases Bert-like model for speech next year (trenton hundred languages on hundred TPUs), which works 50% better, what are we gonna do then? (Nickolay Shmyrev)
    1. Concerns about C++ code will be invalid
    2. Continuous adaptation, life-long learning, few short learning are some things we can keep doing well
    3.  Google can do it. There are lots of people that benefit from it. But also we can solve some other interesting problems. (Sanjeev Khudanpur)
2. One of the reasons image processing became popular was the availability of pretrained models which you could fine-tune on your own data. A similar pitch for K2 would be appreciated and probably why it would become popular with users. (Nagendra Goel)
3. Easy to beat Kaldi using Espnet just using their recipes. (Guoguo Chen)
    * The problem with interfacing k2 with other toolkits is that k2 doesn’t get the brand recognition. The value of kaldi is providing a whole recipe of ASR out-of-box. k2 is just a component for now, k2 should also provide whole recipes.

_<span style="text-decoration:underline;">End-to-end ASR in K2 Q&A </span>_



1. On some public datasets, ESPNet already beats Kaldi models. Is there any plan to compete with that? (Mike Lei)
    * Not as of now, since main focus is K2. (Dan Povey)



---


**Other speech areas**



1. Consider K2 incorporate TTS (Yishay Carmiel)
    1. For broader audience
    2. TTS is quite different and have to think about it  (Dan Povey) 
    3. Lhotse may work better for TTS. (Piotr Zelasko)
2. There are other things such as enhancement, emotion, diarization, etc that can be incorporated by other contributors. (Nagendra Goel)
    * Lhotse will be designed considering all of these. (Dan Povey)



---


**General Q&A (Replies by Dan Povey)**



1. Any plans for making K2 more friendly for deployment on mobile platforms?
    * We can export to onnx will make it easier. It will take a while.
2. Will K2 use components from previous kaldi
    * Mostly from scratch (feature extraction is in Python, maybe the decoder)
3. Kaldi uses a lot of config files. What is your plan for K2?
    *  getting rid of these.
4. Are you going to put an effort into having a low latency decoder?
    * GPU stuff is good for fast batch things. We need to figure out low latency.
5. Adaption (domain,accent,etc), both AM and LM is very important for industry applications, will k2 have any plan to support this?
    * Pretty easy in pytorch to do adaptation. Since k2 is based on pytorch it should be a little easier.
6. Is there going to be backward compatibility with decoders?
    * Depends on how k2 is done, but probably not
7. Where will arpa slm models and pronunciation lexicon fit within k2
    1. Using FSA will make LM naturally fit.
    2. For pronunciation lexicon you can have CTC as an option 
8. Would be good to have something like Montreal forced aligner when you have text and speech but no good transcription.
    * We have to look at it separately. We will get there at some point.
9. Any plans to do full decoding directly on a GPU with lattice generation?
    * K2 will be naturally GPU, both the neural network and the graph search is on GPU.
10. Will K2 support unsupervised training such as Wav2Letter and Wave2vec?
    1. Yes, at least easier to do. We have to see if it is useful.
    2. We will also support pre-trained models.
11. Any plan on supporting privacy compliance, adding support for encrypted decoding, or its way beyond the scope?
    * It is probably not possible. 





