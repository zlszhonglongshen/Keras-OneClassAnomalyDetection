# Keras-OneClassAnomalyDetection
Learning Deep Features for One-Class Classification(AnomalyDetection).  
  
**[Jan 06, 2019] Start of work. Under construction.**  

# Introduction
This repository was created inspired by **[Image abnormality detection using deep learning ーPapers and implementationー - Qiita - shinmura0](https://qiita.com/shinmura0/items/cfb51f66b2d172f2403b)**.  
I would like to express my deepest gratitude for having pleasantly accepted his skill, consideration and article quotation.  
His articles that were supposed to be used practically, not limited to logic alone, are wonderful.  
However, I don't have the skills to read papers, nor do I have skills to read mathematical expressions.  
I only want to verify the effectiveness of his wonderful article content in a practical range.  

# Translating shinmura0's article
## 1. Introduction
There are many methods such as methods using **"[Implemented ALOCC for detecting anomalies by deep learning (GAN) - Qiia - kzkadc](https://qiita.com/kzkadc/items/334c3d85c2acab38f105)"** and methods using **"[Detection of Video Anomalies Using Convolutional Autoencoders and One-Class Support Vector Machines (AutoEncoder)](http://cbic2017.org/papers/cbic-paper-49.pdf)"** for image anomaly detection using deep learning.  
Here is an article on detecting abnormality of images using "Variational Autoencoder".  
**[Image abnormality detection using Variational Autoencoder (Variational Autoencoder) - Qiita - shinmura0](https://qiita.com/shinmura0/items/811d01384e20bfd1e035)**  

The method to be introduced this time is to detect abnormality by devising the loss function using normal convolution neural network(CNN).  
  
「Learning Deep Features for One-Class Classification」 (Subsequent abbreviations, DOC)  
**arxiv：　https://arxiv.org/abs/1801.05365**  
  
![01](media/01.png)  
  
In conclusion, it was found that **this method has good anomaly detection accuracy** and visualization of abnormal spots is also possible.  

## 2. Overview
This paper states that it achieved state-of-the-art at the time of publication.  
In the figure below, we learned under various conditions using normal CNN and visualized the output from the convolution layer with t-SNE.  
  
![02](media/02.png)
  
- Figure (b): Alexnet's learned model with Normal and Abnormal distributed
- Figure (c): Diagram learned with Normal vs Abnormal
- Figure (e): Proposed method (DOC)

I think that it is not only me that thinking that abnormality can be detected also in figure (b).  
However, it is somewhat inferior to figure (e).  
  
In the thesis, it finally uses "k neighborhood method" in (e) to detect abnormality.  
As a learning method, view the images that you want to detect abnormality at the same time, completely different kinds of images, and narrow down the range of the images for which you want to detect anomalies.  

## 3. Preparing data
For learning, prepare the following data.  

|Dataset name|Contents|Concrete example|Number of classes|
|:--|:--|:--|:-:|
|Target data|Image you want to detect abnormality|Product etc.|1|
|Reference data|A data set not related to the above|ImageNet and CIFAR-10|10 or 1,000 or more|

## 4. Preparing the model
![03](media/03.png)  

- The deep learning model **`g`** prepares a learned model.
- In the paper, **`g`** uses Alexnet and VGG16. **`h`** is 1,000 nodes for ImageNet, 10 nodes for CIFAR-10.
- During learning, **`g`** and h of Reference Network (R) and Secondary Network (S) are shared.
- Also, during learning, the weights are fixed except for the last four layers.

## 5. Learning phase

- First, using the reference data, let **`R`** compute the loss function <img src="https://latex.codecogs.com/gif.latex?l_D" />.
- Next, using the target data, let **`S`** calculate the loss function <img src="https://latex.codecogs.com/gif.latex?l_C" />.
- Finally, let 's learn **`R`** and **`S`** at the same time by <img src="https://latex.codecogs.com/gif.latex?l_D" /> and <img src="https://latex.codecogs.com/gif.latex?l_C" />.
  
Total Loss <img src="https://latex.codecogs.com/gif.latex?L" /> is defined by the following formula.  
  
<img src="https://latex.codecogs.com/gif.latex?{L=l_D&plus;\lambda&space;l_C&space;}" />  
  
<img src="https://latex.codecogs.com/gif.latex?l_D" /> uses the cross entropy used in normal classification problems.  
Also in the paper <img src="https://latex.codecogs.com/gif.latex?\lambda=0.1" />.  
  
The most important compact loss <img src="https://latex.codecogs.com/gif.latex?l_C" /> is calculated as follows.  
Let <img src="https://latex.codecogs.com/gif.latex?n" /> be the batch size and let <img src="https://latex.codecogs.com/gif.latex?x_i\in&space;R^k" /> be the output (k dimension) from **`g`**. Then define <img src="https://latex.codecogs.com/gif.latex?z_i"/> as follows.  
  
<img src="https://latex.codecogs.com/gif.latex?{z_i&space;=&space;x_i&space;-&space;m_i}" />
<img src="https://latex.codecogs.com/gif.latex?m_i&space;=&space;\frac{1}{n-1}\sum_{j\not=i}x_j" />
  
<img src="https://latex.codecogs.com/gif.latex?m_i" /> is the average value of the output except <img src="https://latex.codecogs.com/gif.latex?x_i" /> in the batch. At this time, <img src="https://latex.codecogs.com/gif.latex?l_C" /> is defined as follows.  
  
<img src="https://latex.codecogs.com/gif.latex?{l_{C}=\frac{1}{nk}\sum_{i=1}^nz_i^Tz_i&space;}" />

<hr />

**＜Annotation＞**  
As an image, (Strictly speaking it is different) <img src="https://latex.codecogs.com/gif.latex?l_C" /> can be regarded as the variance of the output within the batch.  
When assembling <img src="https://latex.codecogs.com/gif.latex?l_C" /> code, it is troublesome to write "average value other than <img src="https://latex.codecogs.com/gif.latex?x_i" />", I used the following formula in the appendix of the paper.  
  
<img src="https://latex.codecogs.com/gif.latex?l_{C}=\frac{1}{nk}\sum_{i=1}^n\frac{n^2\sigma^2_i}{(n-1)^2}" />  
<img src="https://latex.codecogs.com/gif.latex?\sigma^2_i=[x_i-m]^T[x_i-m]" />  
  
However, <img src="https://latex.codecogs.com/gif.latex?m" /> is the average value of the output within the batch.  
  
<hr />
  
And at the time of learning, I will let you learn so that <img src="https://latex.codecogs.com/gif.latex?l_C" />, which is the variance of the output, also decreases with cross entropy <img src="https://latex.codecogs.com/gif.latex?l_D" />.  
The learning rate seems to be <img src="https://latex.codecogs.com/gif.latex?5\times10^{-5}" />, and the weight decay is set to 0.00005.  
