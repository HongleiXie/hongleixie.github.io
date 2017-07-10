---
layout: post
title:  "NLP: Get hands dirty with Word2Vec"
date:   2017-07-10
---

<span class="dropcap">L</span>ast time I wrote a [post](http://hongleixie.github.io/blog/NLP-words-cooc/) about words co-ocurrences matrix: basically why do we need it and how to create it, along with taking advantage of SVD to reduce dimensions. In this quick post, I will go directly to the implementations of **Word2Vec** in the package `gensim`. No math, no lengthy formulas, as I believe there are tons of useful resources online regarding the `Skip-Gram Model` and `Continuous Bag-of-Words Model` inside and out. Here is someone's [collection](http://mccormickml.com/2016/04/27/word2vec-resources/) of links and notes, all about **Word2Vec**. That's why this post is only focusing on getting some hands-on practice in **Word2Vec**.


### Word2Vec implementations

**Word2Vec** is part of the `gensim` package, and it is only one of many possible embeddings. You can find the documentation [here](https://radimrehurek.com/gensim/models/word2vec.html)

```python
import gensim.models.word2vec as w2v

# load data
books = [open('../data/dorian.txt','r', encoding = 'utf8'),open('../data/earnest.txt','r'),
       open('../data/essays.txt','r'),open('../data/ghost.txt','r'),
       open('../data/happy_prince.txt','r'),open('../data/house_pomegranates.txt','r'),
       open('../data/ideal_husband.txt','r'),open('../data/intentions.txt','r'),
       open('../data/lady_windermere.txt','r'),open('../data/profundis.txt','r'),
       open('../data/salome.txt','r'),open('../data/soul_of_man.txt','r'),
       open('../data/woman_of_no_importance.txt','r')]

# make it as a large corpus
corpus = " ".join([book.read() for book in books])     
```

Same as last time, we use package `nltk` to help us create sentences.

```python
from nltk import sent_tokenize
raw_sentences = sent_tokenize(corpus)

# print one of the sentences
raw_sentences[5005]
```
Randomly print out any one sentence. What I got is as follows: `"Everybody I know says you are very wicked," cried the old lady, shaking her head.`

To create a model we need to feed it a list of sentences, of importance for us within the signature are

- sentences = List of sentences, where each sentence is a list of tokens (words).
- size: Which corresponds to the embedding dimension, default = 100.
- window: The number of words before and after, default = 5. (8 works better according to the paper)
- min_count: Ignores words that appear less than this number where the choice is depending on how large is your training document

So now we need to make each raw sentence into a list of tokens.

```python
from nltk import word_tokenize
sentences = []
for sentence in raw_sentences:
    sentences += [word_tokenize(sentence)]

# sentences is a list of list    
sentences[20]
# you should get:
# ['No', 'artist', 'has', 'ethical', 'sympathies', '.']
```

Alright, everything is ready, let's train our first model!

```python
model_1 = w2v.Word2Vec(sentences = sentences, size = 40, window = 8)
```

Let's check out any random word, for say, let's see how the model translates word 'love' into a 40-dimension vector.

```python
# convert love to size 40 vectors
model_1['love']
```
What if you specify some word not existing in the document? The short answer is that you will get errors, however, there are some parameters you can specify, basically looking for the most similar word's numeric representations, to avoid this kind of errors. Also one thing you need to remember is that they are case sensitive. Let's prove it by the following codes.

```python
# case sensitive
model_1['Love'] == model_1['love']
```

Test the model on the classic example.

```python
model_1.most_similar([model_1['Queen']-model_1['woman']+model_1['man']])
```
It's hardly believe you will end up with word 'King' due to the small training size. However, if we turn to pre-trained **Word2Vec** models, things are much better there.


Visualization is important! Here we are using `tsne` to visualize text data in 2-D space, isn't it amazing?

```python
import numpy as np
from sklearn.manifold import TSNE
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline

vocab = list(model_1.wv.vocab)
X = model_1[vocab]

tsne = TSNE(n_components=2)
X_tsne = tsne.fit_transform(X)

df = pd.concat([pd.DataFrame(X_tsne),
                pd.Series(vocab)],
               axis=1)

df.columns = ['x', 'y', 'word']
df = df.query("word in ('King', 'Queen', 'man', 'woman')")

fig = plt.figure()
ax = fig.add_subplot(1, 1, 1)

ax.scatter(df['x'], df['y'])
for i, txt in enumerate(df['word']):
    ax.annotate(txt, (df['x'].iloc[i], df['y'].iloc[i]))

```
<figure>
    <img src="{{ '/assets/img/Jul10_word2vec.png' | prepend: site.baseurl }}" alt="">
    <figcaption>Word2Vec Visualization</figcaption>
</figure>


### Use GoogleNews pre-trained model
There are many **Word2Vec** pre-trained models out there, here I am only demonstrating GoogleNews one. The pre-trained model can be downloaded from the official [website]([https://code.google.com/archive/p/word2vec/]).

```python
from gensim.models.keyedvectors import KeyedVectors
model_2 = KeyedVectors.load_word2vec_format('../data/GoogleNews-vectors-negative300.bin.gz', binary = True)

model_2.most_similar([model_2['Queen']-model_2['woman']+model_2['man']])
```
