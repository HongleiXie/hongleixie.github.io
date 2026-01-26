---
layout: post
title:  "Floating Point Computations Errors"
date:   2026-01-19
---

<span class="dropcap">F</span>loating-point computations are well-known for their susceptibility to round-off errors. In this post, I aim to document a couple of scenarios where these errors occur and explore potential workarounds where applicable.

## Scenario 1
When the components of a vector are large and slightly different.
```python
import numpy as np
a = np.array([12345], dtype='float32')
b = np.array([12343], dtype='float32')

diff = (a * a).sum() + (b * b).sum() - 2 * np.dot(a, b)
# (a-b)^2 = a^2+b^2-2*a*b = (12345-12343)^2 = 2^2 = 4
# diff = 0 instead of the expected 4
```

Workaround: center the vector components, which doesn’t change the distance.

## Scenario 2
If some components are much larger than others, then during accumulation of distances, smaller components may be cancelled out. Unfortunately I don't find workaround.

```python
import numpy as np

a = np.array([0, 0], dtype='float32')
b = np.array([0, 10000], dtype='float32')
c = np.array([1, 10000], dtype='float32')

diff_ab = ((a - b) ** 2).sum()
diff_ac = ((a - c) ** 2).sum()

# diff_ab = diff_ac = 100000000.0 which is not true
```
