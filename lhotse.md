# Lhotse - audio data preparation toolkit

Lhotse is a next-generation Python library aiming to make speech and audio data preparation flexible and accessible to a wider community.
Like Kaldi, it provides standard data preparation recipes, but extends that with a seamless PyTorch integration through task-specific Dataset classes. 
The data and meta-data are represented in human-readable text manifests and exposed to the user through convenient Python classes.

Lhotse introduces the notion of audio cuts, designed to ease the training data construction with operations such as mixing, truncation and padding that are performed on-the-fly to minimize the amount of storage required. 
Data augmentation and feature extraction are supported both in pre-computed mode, with highly-compressed feature matrices stored on disk, and on-the-fly mode that computes the transformations upon request. 
Additionally, Lhotse introduces feature-space cut mixing to make the best of both worlds.

📄 Documentation: https://lhotse.readthedocs.io/en/latest/

⌨️ Code: https://github.com/pzelasko/lhotse
