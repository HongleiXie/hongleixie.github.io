---
layout: post
title:  "Learning from Imbalanced Data"
date:   2015-12-07
---

<span class="dropcap">I</span> once gave a short talk during lab's Machine Learning seminar regarding classification algorithms in imbalanced data. Technically speaking, any data set that exhibits an unequal distribution between its classes can be considered imbalanced.(He H, Garcia E A.2009)

## Problem
You may naturally ask what's the consequence if you train on imbalanced data simply using the commonly used algorithms? For example, logistic regression, regularized generalized linear models, decision trees even random forest. The resulting problem is that it tends to provide a severely imbalanced degree of accuracy, which means that the majority class has high accuracy rate, like 80%, while the minority has low accuracy rate, usually less than 10%. The reason why these commonly used classification algorithms do not work well because they aim to minimize the overall error rate rather than paying attention to the minority class. This can be a serious problem if the distribution of misclassification cost is unequal. For example, rare cancer diagnosing. The cost of classifying a cancer patient, which is the minority class, to non-cancer is much higher than the other way around. Another example includes credit default classification. 
## Solutions
- Sampling Methods: down-sampling majority class or over-sampling
minority or both. The idea of over-sampling is quite intuitive. Suppose when you are facing with imbalanced data, one way to balance data is adding more samples from minority class. Instead of oversampling minority, another way to balance data is down-sampling majority class
- Cost Sensitive Methods: assigning a high cost (i.e. weight) to
misclassification of minority class and minimizing the overall error rate. However,  the weights are generally hard to choose.

### SMOTE
Synthetic Minority Over-sampling TEchnique (SMOTE) is one of the popular over-sampling algorithm to address imbalanced data problem. The basic idea is to create artificial data based on the feature space similarities between existing minority samples.
<p>Specifically, for each sample in the minority class, consider the K-nearest
neighbors where K is some specified integer. The K-nearest neighbors are defined as the K elements of \\(X_i\\) 
whose Euclidean distance between itself and \\(X_i\\) under consideration exhibits the smallest magnitude along the n-dimensions of feature space \\(X\\).  
To create a synthetic sample, randomly select one of the K-nearest neighbors, then multiply the corresponding feature vector difference with a random number \\(\delta\\) between 0 and 1; and finally, add this vector to \\(X_i\\), allowing the classifiers to better predict unseen samples belonging to the minority class. SMOTE can be implemented in ```DMwR``` package. Let's see an example below. </p>

```r
library(DMwR);
SMOTE(y ~ . ,
    data = train.data,
    perc.over = 200, #number of extra oversampling cases
    k = 5, #number of nearest neighbours
    perc.under = 150, #downsampling
    learner = randomForest,
    mtry = 2,
    ntree = 1e5,
    ...
    );
```

The ```learner``` parameter enables you to specify base classification method and associated parameters (e.g. ```mtry```, ```ntree```).
