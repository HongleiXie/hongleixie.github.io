---
layout: post
title:  "Example Post Formatting"
date:   2014-12-15
---

<p class="intro"><span class="dropcap">C</span>urabitur blandit tempus porttitor. Nullam quis risus eget urna mollis ornare vel eu leo. Vestibulum id ligula porta felis euismod semper. Donec sed odio dui. Aenean lacinia bibendum nulla sed consectetur. test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test test </p>

R code:

```r
## Prepare data
x <- as.data.frame(replicate(20, rnorm(700, sd=10)));
y <- rowSums(x) + rnorm(700, mean=0, sd=5);
train.data <- cbind(x,y);
```

# Heading 1

First level header
==================

Second level header
------

<h3>H3 header</h3>
<h4>H4 header</h4>
<h5>H3 header</h5>
<h6>H6 header</h6>

## Unordered List
* List Item
* Longer List Item
  * Nested List Item
  * Nested Item
* List Item

## Ordered List
1. List Item
2. Longer List Item
    1. Nested OL Item
    2. Another Nested Item
3. List Item

## Definition List
<dl>
  <dt>Coffee</dt>
  <dd>Black hot drink</dd>
  <dt>Milk</dt>
  <dd>White cold drink</dd>
</dl>

