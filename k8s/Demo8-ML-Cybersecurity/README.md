
## Getting to know Python's libraries

Python libraries that are among the most well known and widespread in the field of ML:

   - NumPy 
   - pandas 
   - Matplotlib 
   - scikit-learn
   - Seaborn


### DL libraries
Python libraries for AI, in particular, to exploit the potential of deep learning. The libraries that are as follows:

   - TensorFlow
   - Keras
   - PyTorch


## Types of machine learning used with Cybersecurity           
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

### How to find useful sources of data

In the case of anomaly detection, for example, particular attention must be paid to the data being analyzed. An effective anomaly detection activity presupposes that the training data does not contain the anomalies sought, but that on the contrary, they reflect the normal situation of reference.

If, on the other hand, the training data was biased with the anomalies being investigated, the anomaly detection activity would lose much of its reliability and utility in accordance with the principle commonly known as GIGO, which stands for garbage in, garbage out.

Given the increasing availability of raw data in real time, often the preliminary cleaning of data is considered a challenge in itself. In fact, it's often necessary to conduct a preliminary skim of the data, eliminating irrelevant or redundant information. We can then present the data to the algorithms in a correct form, which can improve their ability to learn, adapting to the form of data on the basis of the type of algorithm used.

For example, a classification algorithm will be able to identify a more representative and more effective model in cases in which the input data will be presented in a grouped form, or is capable of being linearly separable. In the same way, the presence of variables (also known as dimensions) containing empty fields weighs down the computational effort of the algorithm and produces less reliable predictive models due to the phenomenon known as the curse of dimensionality.

This occurs when the number of features, that is, dimensions, increases without improving the relevant information, simply resulting in data being dispersed in the increased space of research.

Also, the sources from which we draw our test cases (samples) are important. Think, for example, of a case in which we have to predict the mischievous behavior of an unknown executable. The problem in question is reduced to the definition of a model of classification of the executable, which must be traced back to one of two categories: genuine and malicious.

To achieve such a result, we need to train our classification algorithm by providing it with a number of examples of executables that are considered malicious as an input dataset.


Quantity versus quality

When it all boils down to quantity versus quality, we are immediately faced with the following two problems:

    What types of malware can we consider most representative of the most probable risks and threats to our company?
    How many example cases (samples) should we collect and administer to the algorithms in order to obtain a reliable result in terms of both effectiveness and predictive efficiency of future threats?

The answers to the two questions are closely related to the knowledge that the analyst has of the specific organizational realm in which they must operate. 

All this could lead the analyst to believe that the creation of a honey-pot, which is useful for gathering malicious samples in the wild that will be fed to the algorithms as training samples, would be more representative of the level of risk to which the organization is exposed than the use of datasets as examples of generic threats. At the same time, the number of test examples to be submitted to the algorithm is determined by the characteristics of the data themselves. These can, in fact, present a prevalence of cases (skewness) of a certain type, to the detriment of other types, leading to a distortion in the predictions of the algorithm toward the classes that are most numerous, when in reality, the most relevant information for our investigation is represented by a class with a smaller number of cases.

In conclusion, it will not be a matter of being able to simply choose the best algorithm for our goals (which often does not exist), but mainly to select the most representative cases (samples) to be submitted to a set of algorithms, which we will try to optimize based on the results obtained.


