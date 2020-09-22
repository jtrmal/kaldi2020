# Kald ASR: Research and Academic Users -- Summary of the meeting #

* Preared by Desh Raj, Yiming Wang, Jonathan Chang, Paola Garcia
* September 17, 2020.

## Introduction (Sanjeev Khudanpur) ##

The townhall was planned to discuss the future of Kaldi. Sanjeev Khudanpur gave an introduction to the evolution of the toolkit.  Kaldi still remains one of the leading tools for researchers and small companies. Part of its success relies on the community contribution to maintain Kaldi up to date. Today, Kaldi is preparing major revisions such as pytorch-ification as the core neural components, automatic differentiation through the FST, and streamlining for data ingestion. The main questions to continue the project are what, how, and who. We will need the help of the community.  

## K2 (Dan Povey) ##
Dan Povey explained his vision of the next Kaldi.  The overall plan is to have two main projects: K2 for the sequence modeling, Lhotse for the data preparation, but also to make sure it works with other platforms. The recipes’ structure will be decided later.

The current Kaldi is based on C++ and bash scripts. The neural network framework is not powerful enough for the new PyTorch and TensorFlow, and it is becoming complex. The Kaldi contributors started a prototype for building a python wrapper, but it was difficult to maintain, so it was abandoned. The idea, now, is to start from scratch. 

### Design Considerations ###

Kaldi new design will have separate packages for data preparation, training, etc, plus small and more maintainable projects. It will also work well with frameworks like PyTorch or Tensorflow. It will avoid the mismatch between training and inference codebases.

### Goals ###

* Make FSA compatible with Pytorch and Tensorflow
* FSAs are differentiable,  have fast GPU implementation,  are compatible with PyTorch; “ragged tensors”. 
* FSA support standard operations: pruned composition, determinization, n-best rescoring
* View CTC and LF-MMI as FSA
* Can use OpenFST but will eventually implement it natively

### K2 implementation ###
K2 will be mostly C++/Cuda
The workhorse in K2 will be ``RaggedTensor'': arbitrarily nesting vector<vector<int>>
It will have dependencies on cub, DLPack, pybind11

### Why not Pytorch or Tensorflow ###
The new Kaldi will not be PyTorch or TensorFlow because it is not possible to efficiently implement some algorithms like composition and determinization. TensorFlow RaggedTensor codebase is too complex and not integrable with PyTorch. Autograd can be used since it allows the algorithms to be pure FSAs, and it is implemented at Python level.


### Code ###
The Kaldi contributors need to rethink algorithms for GPU implementation (because of loops). There will not be a lot of GPU specific code; most C++ lambdas are CPU-based, so that reusable code is accessible for more developers. Operations such as pruned composition, computing expectations are now on GPU; the determinization will come later.
 
### Timeline ###
We plan to release the code Mid-to-late October. The initial release will be CTC and LF-MMI. By the end of the year, non-trivial uses such as multi-pass jointly optimized systems will be implemented.  Later on, the contributors will focus on recipes (still undecided structure), modeling and Lhotse.   

## Lhotse (Piotr Zelasko) ##
Kaldi was designed with experts in mind, not easy-to-use. Lhotse proposes to make things accessible to a broader community. 
 
### Key aspects ###

* Python-centric: numpy, pandas, etc.
* Easy-to-manipulate data structures
* Data prep for standard speech corpora; e.g. SWBD, AMI, mini-Librispeech
* For now PyTorch but later TensorFlow
* Main idea: cuts (arbitrary segments), supervision (similar to segments - use of metadata).


## Panel Discussion: Kaldi in education and research (Moderated by Jan "Yenda" Trmal  ##

### Kaldi remarks/opinions ###

* ASR is a difficult subject - Yenda
* Kaldi is hard for students/beginners (experts know how to use it) - John Hansen, Eric Fosler, Hermann Ney, Peter Bell, Thomas Hain
* Kaldi is trying to be too many things: be cutting edge as well as being accessible as well as commercial - Eric Fosler
* Kaldi is a research tool, there are other tools to teach the basics - Hermann Ney
* Multi-task training (or anything non-standard with neural networks) is complicated with Kaldi so people move to PyTorch - Peter Bell
* All recipes (SWBD, Librispeech, etc.) are data-oriented rather than algorithm oriented - Thomas Hain
* One of the successes of Kaldi is the recipes (clone and run recipes) - Lukas Burget
* Glad that the community is still continuing in this area and not just in E2E - Peter Bell


### Research goals ###

* Analysis and debugging capability needed - Chin-Hui Lee
* Plug-in Interface with other tools - Chin-Hui Lee
* Visual Interfaces for every module- Chin Hui Lee
* For DNNs Reinhold Haeb-Umbach
* Go Beyond ASR, machine translation for example - Ahmed Ali
* Able to handle more languages and low resource languages - Sakriani Sakti
* Kaldi needs scaffolding, different people taking on different parts -Eric Fosler, SK, Yenda
* Train on infinite large datasets - Reinhold Haeb-Umbach
* Separate data preparation and modeling - Reinhold Haeb-Umbach
* Perform augmentation on the fly - Reinhold Haeb-Umbach
* Make it easy to back-propagate from the acoustic model to the front-end - Reinhold Haeb-Umbach
* Able to adapt models to speech changes (patient monitoring) - Emily Mover
* Handle under-represented speech from disease patterns - Emily Mover 
* Consider in the design that not everybody has access to multiple GPU’s - Sakriani Sakti
* Accommodate self-supervised pre-training - Sakriani Sakti
* Able to do adaptation etc. (not straightforward in LF-MMI in Kaldi) - Peter Bell
* Make things more algorithm oriented, rather than data-oriented - Thomas Hain, Peter Bell, Eric Fosler
* Have more modular systems instead of huge complex systems - Emily Mover
* Downstream integration with things like chat bots - Eric Fosler
* Design based on algorithms in the literature  (pointers to papers)- Lukas Burget
* Able to be interfaced with other projects -Lukas Burget
* Make things more explicit in recipes instead of code -Lukas Burget
* Sustainable science!!  We can't compete with some of the research groups with more resources. What we can do is still contribute to science by making a change in a meaningful way - Thomas Hain

### Education goals ###

* Create problems for teaching signal processing or ML -Mark Hasegawa
* Easy install “pip install k2”  -Mark Hasegawa
* Easy to teach students how to implement a hybrid ASR  -Mark Hasegawa
* build ASR from scratch; download and easy install, e.g. in Google colab -Eric Fosler
* Create smaller examples integrable into classroom teaching- John Hansen, SK
* Provide generic examples for building recipes -John Hansen
* visualization tools particularly for teaching purposes - Reinhold Haeb-Umbach
* New people want deep neural nets to solve everything. Kaldi needs easy onboarding examples; requires some soul-searching (can’t be everything to everyone) - Alex Weibel
* Instantly available recipes, easy to use, e.g. digit recognition - John Hansen, Emily Mover
* Able to Build small footprint tasks, for low resource languages, keyword search and people from other fields - John Hansen, Peter Bell
* Build examples of manageable/small code:
* Fearless Steps (100h of speech): independent tasks within the corpus can be identified -John Hansen
* Common Voice (Mozilla): include a layer that interfaces with Common Voice (or other corpora) to easily get data from different languages - Ahmed Ali 
* Think about how to get users from outside the small niche of speech researchers so that it can work as a black box speech recognizer ( ASR for dialog, psychologists,etc) - Alex Weibel, John Hansen
* Students need instant gratification and make things fun - Alex Weibel
* Example: Click on the webpage, you don’t have to prepare anything - Alex Weibel.
* Need to involve the students in the process. Actively ask the students to create pull requests - Julia 
* Invite students as contributors - Yenda 
* Students should be able to use it easily so that they can feel confident and learn - Emily Mover

### Commitments/plans ###

* Planning to use pip to easy install - Dan Povey
* Hopefully, in a month we will have the basics running - Dan Povey
* Kaldi 2 will try to be cutting edge (research), accessible (education), commercial (companies) - Dan Povey
* Support multichannel- Dan Povey
* Data augmentation on the fly should be possible in Lhotse - Dan Povey
* Recipes are TBD - Dan Povey
* Current Kaldi will be maintained - Dan Povey
* Plan to have a similar amount of coverage and to have all of the important aspects of the recipes. - Dan Povey
* Lhotse might be organized by subject instead of by database - Dan Povey
*  The hope is that we won't even need context (or transition probabilities, trees) because some of these neural network things can deal with it - Dan Povey 
* Reproducible results with recipes will happen - SK
*  We  hope that Kaldi 2 will have better performance for specific hardware, but it might take a while to implement - Dan Povey
* The main avenue to contribute to K2 is pull requests. We are open to contributions! - Dan Povey
* K2 will never support the “exact” same features as the previous version.  We estimate to have the same performance in February or March 2021  - Dan Povey
*  Generally aiming for top performance, flexibility and producibility - Dan Povey
* We are creating jupyter notebooks in Lhotse to demonstrate usage examples -Piotr Zelasko


### Some other ideas ###

* Teachers can have in-class contests building a system using Kaldi that will contribute to the repository. -SK
* Probably Kaldi should decide on industry, research, teaching -Alex Weibel





