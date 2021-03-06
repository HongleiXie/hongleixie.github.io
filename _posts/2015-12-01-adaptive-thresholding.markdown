---
layout: post
title:  "Adaptive Thresholding"
date:   2015-12-01
---
<span class="dropcap">M</span>otivated by applications in a wide range of fields in signal processing, social science, finance, genetics etc, statistical inference in high dimensional data is a problem of great
interest. Covariance matrix plays an important role in many fundamental models including
principal component analysis and linear/quadratic discriminant analysis. In high dimensional
data setting, we often observe sparse covariance matrix. For example, in cancer microarray experiment,
it has been assumed that most of pairs of gene expressions are insignificant. Thus, efforts
should be made to take advantage of the sparsity assumption when estimating covariance matrices.

## Very Brief Intro ##
<p>
Let \(X = (X_1, X_2, ...X_p)^T\) be a \(p\)-variate vector with true covariance matrix \(S_0\). Given i.i.d random
sample \(X_1, X2, ...X_n\) from the distribution of \(X\), the sample covariance matrix is given by:
\[
\Sigma_n = (\hat{\sigma}_{ij} )_{p\times p} = \frac{1}{n-1} \sum_{k=1}^{n} (\mathbf{X}_k- \mathbf{\bar{X}})(\mathbf{X}_k- \mathbf{\bar{X}})^T
\]
</p>
However, we know that sample covariance is not a consistent estimator. There have been a vast of literature discussing estimation on sparse covariance matrix. A popular approach to introduce sparsity is to add penalty (i.e. thresholding methods) on sample covariance matrix. 

The idea is as simple as this: we all know sample covariance is not a good choice, but it's our only available choice. At least it's a good *sample* to start with. The rest of thing is modification, making it behaving well theoretically. 

Many thresholding methods have been proposed which can be loosely divided into two classes: universal thresholding and adaptive thresholding. Universal thresholding means that there is a single threshold for all entries while the adaptive thresholding methods use entry dependent thresholds. It is clear that sample covariance matrix might have a wide range of
variability therefore universal thresholding may not be good in such scenarios. Related theories and simulation studies have showed adaptive thresholding estimator uniformly outperform the universal estimator.

In an addition, there are three commonly used thresholding functions in practice.

- Hard thresholding `\[s^{H}_{\lambda}(z) = zI(|z| > \lambda)\]`
- Soft thresholding `\[s^{S}_ {\lambda}(z) = sign(z)I(|z| - \lambda)_{+}\]`
- Adaptive Lasso thresholding `\[s^{AL}_{\lambda}(z) = z(1 - |\lambda/z|^\eta)_{+})\]`
- Smoothly Clipped Absolute Deviation Penalty (SCAD)

## What I have done ##
Little theoretical work or numerical work has been done for choice between
different thresholding functions. Based on the adaptive thresholding estimation method, I will fill in the gap, comparing performance of different thresholding functions under adaptive estimation framework. Please see the [paper](http://arxiv.org/pdf/1102.2237.pdf) for details of the procedures. 
In an addition, Fan and Li (2001) proposed a method, Smoothly Clipped Absolute Deviation Penalty (SCAD), serves as the compromise between hard and soft thresholding. And the resulting solution is continuous because it will not penalize excessively on large values. We do not include SCAD into comparison in the present essay for simplicity reason.

## Simulation Study ##

### Simulation Settings
<p>
\(\mathbf{\Sigma} = diag(\mathbf{A_1}, \mathbf{A_2})\) where 
\(\mathbf{A_1} = (\sigma_{ij})_{1 \le i,j \le p/2}, \quad \sigma_{ij} = (1- \frac{\left | i-j \right |}{10})_+ 
\mathbf{A_2} = 4\mathbf{I}_{p/2 \times p/2}\)
</p>
<p>
We know the model generates a banded matrix with ordering. 
\(n =\) 100 i.i.d \(p\) - variate random vectors are produced from \(N(\mathbf{0}, \mathbf{\Sigma})\). We choose \(p =\) 30, 100, 200, 500 to represent different scenarios. To avoid the sampling bias, 100 replications are generated under each setting.
</p>

### Numerical Results

The performance is measured by matrix spectral loss and TPR/FPR. Specifically, spectral loss is defined as the largest singular value of the risk.
Support recovery ability can be measured by true positive rate (TPR) and false positive rate (FPR). Note that for the sample covariance, TPR = 1 and FPR = 1.

### Conclusion

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

## What's Next
[Tony Cai](http://www-stat.wharton.upenn.edu/~tcai/) gave us a mathematically fancy method to estimate high dimensional covariance matrix, It's for me more like a mission impossible! Imagine how many parameters you have and how little information you know! (i.e. observations n), plus sparsity pattern, which is our case here) Anyhow, what I do concern is the efficiency when it goes to production. So my next step might be converting a few key (slow) functions to C++.

***If interested, please check out the source R codes in my [Github](https://github.com/HongleiXie/adaptive-thresholding.git).***


