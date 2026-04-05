---
layout: post
title:  "Accelerating LLM Inference - Speculative Decoding"
date:   2026-04-04
---

<span class="dropcap">L</span>LM are incredibly capable, but they are notoriously slow to run. Because they generate text one token at a time, decoding $K$ tokens requires $K$ serial runs of the model. [Speculative Decoding](https://arxiv.org/abs/2211.17192) is a novel algorithm that breaks this bottleneck, allowing for **2X-3X acceleration** without changing the model’s output distribution.

### The Core Idea: Guess and Verify

The insight behind speculative sampling is twofold: first, "easy" language tasks can often be approximated by much smaller, faster models; second, large model inference is usually limited by **memory bandwidth** rather than raw computation. 

By using a small **approximation model** ($M_q$) to "speculate" or guess several tokens in advance, we can then use the large **target model** ($M_p$) to verify those guesses in a **single parallel run**. This increases concurrency and generates multiple tokens per iteration.

### The Mechanics of Speculative Sampling

For every step, the system follows a "guess-and-verify" loop:
1.  **Speculate:** The small model $M_q$ generates a sequence of $\gamma$ tokens autoregressively.
2.  **Verify:** The target model $M_p$ reviews all these guesses simultaneously in one parallel pass.
3.  **Accept/Reject:** Each guess $x$ is accepted if it aligns with the target model’s distribution. Specifically:
    *   If the approximation probability $q(x)$ is less than or equal to the target probability $p(x)$, the token is **always accepted**.
    *   If $q(x) > p(x)$, the token is **rejected** with a probability of $1 - \frac{p(x)}{q(x)}$. In practice, this is implemented by sampling a uniform random variable $r∼U(0,1)$ and accept the guess if $r>\frac{p(x)}{q(x)}$
4.  **Correct:** If a token is rejected, the target model provides a "correction" token sampled from an **adjusted distribution**.

### A Technical Walkthrough: The Correction Step

To see how the math ensures the output remains identical to the large model, let’s look at a concrete example with a four-token vocabulary.

#### The Setup
*   **Prompt:** The Japan
*   **Vocabulary:** `{"is", "stock", "girl", "cherry"}`
*   **Target Model Distribution ($p$):** `[0.4, 0.3, 0.2, 0.1]`
*   **Approx Model Distribution ($q$):** `[0.5, 0.25, 0.15, 0.1]`

#### Scenario: The small model guesses the token **"is"**
*   **The Decision:** Since $q(\text{"is"}) = 0.5$ and $p(\text{"is"}) = 0.4$, the approximation is "overconfident." The system calculates the rejection probability: $1 - \frac{0.4}{0.5} = 0.2$. There is a 20% chance this guess is rejected.

#### The Correction (Sampling from $p'$)
If the guess `"is"` is rejected, the system must sample a new token from the adjusted distribution $p'(x)$, which focuses on the probability mass the small model missed. 

1.  **Calculate Raw Differences:** We find $\max(0, p(x) - q(x))$ for every token in the vocabulary:
    *   "is": $\max(0, 0.4 - 0.5) = \mathbf{0}$
    *   "stock": $\max(0, 0.3 - 0.25) = \mathbf{0.05}$
    *   "girl": $\max(0, 0.2 - 0.15) = \mathbf{0.05}$
    *   "cherry": $\max(0, 0.1 - 0.1) = \mathbf{0}$
2.  **Normalize:** These differences sum to $0.1$. This value is $1 - \beta$, where $\beta$ is the overall acceptance rate. We divide the differences by $0.1$ to create a valid distribution. 
3.  **Final Adjusted Probabilities:** "is" (0), "stock" (0.5), "girl" (0.5), "cherry" (0)
4.  **Result:** The system will sample the replacement token from a 50/50 split between "stock" and "girl."

### Why It Matters

Because this method guarantees that the final token $x$ is distributed according to $p(x)$, it provides a **mathematical guarantee of identical outputs**. You get the power of a massive 100B+ parameter model with the latency benefits of a model a fraction of its size. 

### Choosing the number of speculative tokens to generate
The parameter **$\gamma$** is the number of tokens the small model guesses before the large model intervenes. Choosing the optimal $\gamma$ is critical for maximizing speedup and depends on two factors:
- **$\alpha$ (Acceptance Rate):** How often the target model agrees with the small model.
- **$c$ (Cost Coefficient):** The ratio of the time it takes to run the small model versus the large model.

The **Expected Improvement Factor** is calculated as:

$$\frac{1 - \alpha^{\gamma+1}}{(1 - \alpha)(\gamma c + 1)}$$

