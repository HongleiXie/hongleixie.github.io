---
layout: post
title:  "Adaptive Thresholding"
date:   2015-12-01
---
Motivated by applications in a wide range of fields in signal processing, social science, finance, genetics etc, statistical inference in high dimensional data is a problem of great
interest. Covariance matrix plays an important role in many fundamental models including
principal component analysis and linear/quadratic discriminant analysis. In high dimensional
data setting, we often observe sparse covariance matrix. For example, in cancer microarray experiment,
it has been assumed that most of pairs of gene expressions are insignificant. Thus, efforts
should be made to take advantage of the sparsity assumption when estimating covariance matrices.

## Very Brief Intro ##
<p>
Let \(X = (X_1, X_2, ...X_p)^T\) be a \(p\)-variate vector with true covariance matrix \(S_0\). Given i.i.d random
sample \(X_1, X2, ...X_n\) from the distribution of \(X\), the sample covariance matrix is given by:
$$

$$
</p>
However, we know that sample covariance is not a consistent estimator. There have been a vast of literature discussing estimation on sparse covariance matrix. A popular approach to introduce sparsity is to add penalty (i.e. thresholding methods) on sample covariance matrix. 

The idea is as simple as this: we all know sample covariance is not a good choice, but it's our only available choice. At least it's a good *sample* to start with. The rest of thing is modification, making it behaving well theoretically. 

Many thresholding methods have been proposed which can be loosely divided into two classes: universal thresholding and adaptive thresholding. Universal thresholding means that there is a single threshold for all entries while the adaptive thresholding methods use entry dependent thresholds. It is clear that sample covariance matrix might have a wide range of
variability therefore universal thresholding may not be good in such scenarios. Related theories and simulation studies have showed adaptive thresholding estimator uniformly outperform the universal estimator.

In an addition, there are three commonly used thresholding functions in practice.
<p>
 Hard thresholding: \[s^{Hard}_{\lambda}(z) = zI(|z| > \lambda)\]
 Soft thresholding: \[s^{Soft}_{\lambda}(z) = sign(z)I(|z| - \lambda)_{+}\]
 Smoothly Clipped Absolute Deviation Penalty (SCAD)
</p>

## What I have done ##
Little theoretical work or numerical work has been done for choice between
different thresholding functions. Based on the adaptive thresholding estimation method, I will fill in the gap, comparing performance of different thresholding functions under adaptive estimation framework. Please see the [paper](http://arxiv.org/pdf/1102.2237.pdf) for details of the procedures.

## Simulation Study ##
### Simulation Settings ###

### Numerical Results ###

### Conclusion ###
<p>
From the simulation results, hard thresholding seems to perform the best in terms of loss under
spectral norm. Next comes to the adaptive lasso. The sample covariance, as a baseline estimator,
not surprisingly turns out to be one generating the largest bias. However, when \(p>>n\), the
difference between different thresholding functions are not that large anymore. Another interesting
fact lies in the standard errors: with the increasing \(p\), hard, soft and adaptive lasso thresholding
all exhibit decay pattern in standard errors. However, the loss of sample covariance tends to have
larger variability if \(p\) becomes bigger. As for the support recovery, it appears that no significant
difference between those three thresholding methods. How does it come? Maybe the support recovery ability largely depends on the estimation method where choice of different thresholding functions has very little impact on them?
</p>
## What's Next? ##
Tong Cai gave us a mathematically fancy method to estimate high dimensional covariance matrix, It's for me more like a missing impossible! Imagine how many parameters do you have and how little information do you know! (i.e. observations $n$, plus sparsity pattern, which is our case here) Anyhow, what I do concern is the efficiency when it goes to production. So my next step might be converting a few key(slow) functions to C++.

***Please check out the source R codes in my [Github](https://github.com/HongleiXie/adaptive-thresholding.git).***


