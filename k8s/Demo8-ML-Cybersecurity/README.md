## Types of machine learning            
The process of mechanical learning from data can take different forms, with different characteristics and predictive abilities.In the case of ML (which, as we have seen, is a branch of research belonging to AI), it is common to distinguish between the following types of ML:
- Supervised learning
- Unsupervised learning
- Reinforcement learning
The differences between these learning modalities are attributable to the type of result (output) that we intend to achieve, based on the nature of the input required to produce it.


### Supervised learning

In the case of supervised learning, algorithm training is conducted using an input dataset, from which the type of output that we have to obtain is already known.

In practice, the algorithms must be trained to identify the relationships between the variables being trained, trying to optimize the learning parameters on the basis of the target variables (also called labels) that, as mentioned, are already known.

An example of a supervised learning algorithm is classification algorithms, which are particularly used in the field of cybersecurity for spam classification.

A spam filter is in fact trained by submitting an input dataset to the algorithm containing many examples of emails that have already been previously classified as spam (the emails were malicious or unwanted) or ham (the emails were genuine and harmless).

The classification algorithm of the spam filter must therefore learn to classify the new emails it will receive in the future, referring to the spam or ham classes based on the training previously performed on the input dataset of the already classified emails.

Another example of supervised algorithms is regression algorithms. Ultimately, there are the following main supervised algorithms:

   - Regression (linear and logistic)
   - k-Nearest Neighbors (k-NNs)
   - Support vector machines (SVMs)
   - Decision trees and random forests
   - Neural networks (NNs)
    

### Unsupervised learning

In the case of unsupervised learning, the algorithms must try to classify the data independently, without the aid of a previous classification provided by the analyst. In the context of cybersecurity, unsupervised learning algorithms are important for identifying new (not previously detected) forms of malware attacks, frauds, and email spamming campaigns.

Here are some examples of unsupervised algorithms:

   - Dimensionality reduction:
       - Principal component analysis (PCA)
       - PCA Kernel
   - Clustering:
       - k-means
       - Hierarchical cluster analysis (HCA)


### Reinforcement learning

In the case of reinforcement learning (RL), a different learning strategy is followed, which emulates the trial and error approach. Thus, drawing information from the feedback obtained during the learning path, with the aim of maximizing the reward finally obtained based on the number of correct decisions that the algorithm has selected.

In practice, the learning process takes place in an unsupervised manner, with the particularity that a positive reward is assigned to each correct decision (and a negative reward for incorrect decisions) taken at each step of the learning path. At the end of the learning process, the decisions of the algorithm are reassessed based on the final reward achieved.

Given its dynamic nature, it is no coincidence that RL is more similar to the general approach adopted by AI than to the common algorithms developed in ML.

The following are some examples of RL algorithms:

   - Markov process
   -  Q-learning
   -  Temporal difference (TD) methods
   -  Monte Carlo methods

In particular, Hidden Markov Models (HMM) (which make use of the Markov process) are extremely important in the detection of polymorphic malware threats.

### Algorithm training and optimization

When preparing automated learning procedures, we will often face a series of challenges. We need to overcome these challenges in order to recognize and avoid compromising the reliability of the procedures themselves, thus preventing the possibility of drawing erroneous or hasty conclusions that, in the context of cybersecurity, can have devastating consequences.

One of the main problems that we often face, especially in the case of the configuration of threat detection procedures, is the management of false positives; that is, cases detected by the algorithm and classified as potential threats, which in reality are not (example Fraud Prevention)

The management of false positives is particularly burdensome in the case of detection systems aimed at contrasting networking threats, given that the number of events detected are often so high that they absorb and saturate all the human resources dedicated to threat detection activities.

On the other hand, even correct (true positive) reports, if in excessive numbers, contribute to functionally overloading the analysts, distracting them from priority tasks. The need to optimize the learning procedures therefore emerges in order to reduce the number of cases that need to be analyzed in depth by the analysts.

This optimization activity often starts with the selection and cleaning of the data submitted to the algorithms.
