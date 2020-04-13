---
layout: post
title: "Recommendation Systems for Implicit Feedback Dataset - Part 1"
date: 2018-07-08
---
<span class="dropcap">T</span>his is the first of a series of _non-comprehensive_ paper reviews for the recommendation systems for **implicit** feedback datasets.

### Navigations
- [**Part 1:**](http://hongleixie.github.io/blog/implicit-CF-part1/) Hu, Yifan, Yehuda Koren, and Chris Volinsky. "Collaborative filtering for implicit feedback datasets." _Data Mining, 2008. ICDM'08. Eighth IEEE International Conference on_. Ieee, 2008.
- **Part 2:** Liang, Dawen, et al. "Variational Autoencoders for Collaborative Filtering." _arXiv preprint arXiv:1802.05814_(2018).


### Collaborative filtering for implicit feedback datasets: High-level summary
- Although the classic, elegantly-written paper has been published for nearly 10 years, it still enjoys high-citation until now as it's one of the first papers introducing latent factor algorithm which is especially tailored to implicit feedback recommenders.
- It also provides scalable optimization procedure which is claimed to be scaled **_linearly_** with the data size.
-  Surprisingly, the proposed algorithm allows explaining the recommendation results to the end users, which is a rarity among the latent factor models. This interesting feature would probably play a big role in integrating the AI solution into some traditional enterprise clients.

### Explicit v.s. Implicit feedback
What's the main distinctions between explicit and implicit-feedback recommenders? Why do we have to address the two types of recommenders differently?
- Explicit feedback includes explicit input by users regarding their interest in products. For example, the 5-star rating system in Amazon.
- Implicit feedback, which indirectly reflects opinion through observing user behavior, includes purchase history, browsing history, search patterns, watching habits etc.
- What's the features of _implicit feedback_?
  1. **No negative feedback.** For example, a user that did not watch a certain show might have done so because she dislikes the show or just because she did not know about the show or was not available to watch it. This fundamental asymmetry does not exist in explicit feedback where users tell us both what they like and what they dislike.
  2. **Inherently noisy.** For example, we may view purchase behavior for an individual, but this does not necessarily indicate a positive view of the product.
  3. Numerical values of implicit feedback describe the _frequency_ of actions, e.g., how much time the user watched a certain show, how frequently a user is buying a certain item, etc. A larger value is not indicating a higher _preference_. For example, the most loved show may be a movie that the user will watch only once, while there is a series that the user quite likes and thus is watching every week. The numerical value of the feedback tells us about the _confidence_ that we have in a certain observation.
  4. Evaluation of implicit-feedback recommender requires appropriate measures. In explicit-feedback recommender, we may use `RMSE` or some ranking metrics to measure accuracy of the recommendation system, whereas in implicit feedback setting we need to take into account more factors. For example, it is unclear how to evaluate a show that has been watched more than once, or how to compare two shows that are on at the same time, and hence cannot both be watched by the user.
  5. Explicit ratings are typically unknown for the vast majority of user-item pairs. However, with implicit feedback it would be natural to assign values to all user-item pairs.

### Previous work
Let's describe the models in the TV recommender case. Denote `\(r_{u,i}\)`  as how many times user `\(u\)` fully watched the show `\(i\)`. So `\(r_{u,i} = 0.7\)` indicates that `\(u\)` watched 70% of the show, while for a user that watched the show twice we will set `\(r_{u,i} = 2.0\)`.
As discussed in the  **Explicit v.s. Implicit feedback** distinction #5,  if no action was observed `\(r_{u,i}\)` is set to zero, thus meaning in our examples zero watching time.
#### Neighborhood models
Let's restrict the discussion into item-based neighborhood models as it has better scalability and improved accuracy. Our goal is to compute the similarity of item `\(i\)` and item `\(j\)`, usually based on Pearson correlation coefficient `\(s_{i,j}\)` and then predict `\(r_{u,i} , u \in U, i \in I\)`.

Disadvantages in regards to implicit feedback:
- Some enhancements such as correcting for biases caused by varying mean ratings of different users and items are less relevant to implicit feedback datasets.
- Frequencies for disparate users might have very different scale depending on the application, and it is less clear how to calculate similarities.
- Not providing the flexibility to make a distinction between user _preferences_ and the _confidence_ we might have in those preferences.

Reference paper about the topic:
M. Deshpande, G. Karypis, “Item-based top-N recommendation algorithms”, _ACM Trans. Inf. Syst. 22 (2004) 143-177._

#### Latent factor models
This paper borrowed this approach to implicit feedback datasets, with modifications both in the model formulation and in the optimization technique.

### Model
I will intentionally omit the rationale behind the formulation that you can find in the original paper.
#### Some notations
Let's denote the _preference_ of user `\(u\)` to item `\(i\)` by `\(p_{ui}\)`
`\[
p_{ui} = \begin{cases}
  1, & r_{ui} > 0, \\
  0, & {r_{ui}=0}
\end{cases}
\]`
And `\(c_{ui}\)` measures the confidence in observing `\(p_{ui}\)`. The authors suggested two forms of choices of `\(c_{ui}\)`
`\[
c_{ui} = 1 + \alpha r_{ui}
\]`
or
`\[
c_{ui} = 1 + \alpha log(1 + r_{ui}/\epsilon)
\]`
The intuition behind the formula is that with the increasing of `\(r_{ui}\)`, we have more confidence that the user indeed likes the item. The rate of increase is controlled by the constant `\(\alpha\)`.  In their experiments, setting `\(\alpha = 40\)` was found to produce good results.

#### Loss function
The loss function is similar to matrix factorization techniques which are popular for explicit feedback data, with two important distinctions:
- We need to account for the varying confidence levels by introducing `\(c_{ui}\)`. Also the authors argued that transferring the raw observations `\(r_{ui}\)` into two separate magnitudes with distinct interpretations: preferences and confidence levels is essential to improving accuracy.
- Optimization should account for all possible `\(u, i\)` pairs, rather than only those corresponding to observed data. So it prevents stochastic gradient descent due to the insane computation complexity. Alternating-least-squares (ALS) optimization process was applied in the experiment study.
$$
L = \sum\limits_{u,i } c_{ui}(p_{ui} - \textbf{x}_{u}^{\intercal} \cdot{} \textbf{y}_{i})^{2} + \lambda (\sum\limits_{u} \left\Vert \textbf{x}_{u} \right\Vert^{2} + \sum\limits_{i} \left\Vert \textbf{y}_{i} \right\Vert^{2})
$$


### Experimental study
#### Set-up
- `\(r_{ui}\)` is denoted as, for each user `\(u\)` and show `\(i\)`, how many times user `\(u\)` watched show `\(i\)` (related is the number of minutes that a given show was watched --- for all of our analysis we focus on show length based units).
- Testing dataset is constructed as all channel tune events during the single week following a 4-week training period. And for each user they removed the “easy” predictions from the test set corresponding to the shows that had been watched by that user during the training period. Also, they filtered out all entries with `\(r^{t}_{ui} < 0.5\)` as watching less than half of a show is not a strong indication that a user likes the show.
- Since `\(r_{ui}\)` tends to vary significantly over a large range, so they applied
`\[
c_{ui} = 1 + 40\times log(1 + r_{ui}/10^{-8})
\]`

- They adjusted the _momentum effect_ by for the `\(t\)`-th show after a channel tune, we assign it assigning it a weighting `\(\frac{e^{-(2t-6)}}{1+e^{-(2t-6)}}\)`. To do this, it requires that we need to map every tuning session into a particular show.

#### Evaluation
Essentially, we need to see if the order of recommendations given for each user matches the items they ended up consumed (whether purchasing or watching). Particularly in this use case, the authors also argued that precision based metrics are not very appropriate, as they require knowing which shows are undesired to a user. However, we don't have such data as not watching a show can stem from multiple reasons. While watching a show is an indication of liking it, making recall based metrics applicable.
BTW, I also noticed that some [people](https://jessesw.com/Rec-System/) also used AUC to evaluate the recommender system.

*Notations*

We denote `\(Rank_{ui}\)` as the percentile-ranking of show `\(i\)` within the ordered list of all programs prepared for user `\(u\)`.
So `\(Rank_{ui} = 0\%\)` would mean that show `\(i\)` is predicted to be the **most desirable** for user `\(u\)` while `\(Rank_{ui} = 100\%\)` indicates that show `\(i\)` is predicted to be the **least preferred** for user `\(u\)`, thus placed at the end of the list.

`\[
\overline{Rank} = \frac{\sum\limits_{u,i } r^{t}_{ui} Rank_{ui}}{\sum\limits_{u,i } r^{t}_{ui}}
\]`

Lower values of rank are more desirable, with 50% as the random guessing.

*Results*

To sum up:
- If you want to implement item-based neighborhood methods, try to
  - **(1)** Take all items as “neighbors”, not only a small subset of most similar items.
  - **(2)** Use cosine similarity function.
- Quality of the recommendations can be measured by studying the cumulative distribution function of `\(Rank_{ui}\)`. So what is the distribution of percentiles for the shows that were actually watched in the test set? If the model does well, all of the watched shows will have low percentiles. From the reported figure, we see that a watched show is in the top 1% of the recommendations from the model about 27% of the time.
- Results get much better had they left all previously watched programs in the test set (without removing all user-program events that already occurred in the training period).
- The model becomes much easier to predict popular programs, while it is increasingly difficult to predict watching a non popular show. The similar observations cannot be found in the user side though.
- Train the model under the **full** loss function.
- A possible extension of the model –-- adding a dynamic time variable addressing the tendency of a user to watch TV on certain times.

### Just a few lines of Python implementation
```python
import implicit
alpha = 40 #as suggested by the paper
user_vecs,item_vecs = implicit.alternating_least_squares((train_data*alpha).astype('double'),
                                                          factors = 20,
                                                          regularization = 0.1,
                                                          iterations = 50)
full_matrix = item_vecs.dot(user_vecs.T) #predicted r_{ui}                                              
```
