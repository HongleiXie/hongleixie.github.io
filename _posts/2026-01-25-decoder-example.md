---
layout: post
title:  "Transformer -- Decoder-Only Model Explained In Codes"
date:   2026-01-25
---

<span class="dropcap">T</span>his is the very first post of many Transformer series posts. Keeping track of my own learning notes.

## Two types of model classes
I will mainly use `transformers` library from HuggingFace. The `transformers` library has two types of model classes:  `AutoModelForCausalLM`  and `AutoModelForMaskedLM`. 
Causal language models represent the **decoder-only** models that are used for text generation. They are described as causal, because to predict the next token, the model can only attend to the preceding left tokens. 
Masked language models represent the **encoder-only** models that are used for rich text representation. They are described as masked, because they are trained to predict a masked or hidden token in a sequence.

## Code example
Let's generate a single token to a prompt using a light-weight decoder-only LLM `microsoft/Phi-3-mini-4k-instruct`.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
# Load model and tokenizer
tokenizer = AutoTokenizer.from_pretrained("microsoft/Phi-3-mini-4k-instruct")

model = AutoModelForCausalLM.from_pretrained(
    "microsoft/Phi-3-mini-4k-instruct",
    device_map="cpu",
    torch_dtype="auto",
    trust_remote_code=True,
)
```
> Note that if you receive a warning that the `flash-attention` package is not found. That's because flash attention requires certain types of GPU hardware to run. Since we are not using any GPU, you can ignore this warning

Now let's print the model to take a look at its architecture:
```
Phi3ForCausalLM(
  (model): Phi3Model(
    (embed_tokens): Embedding(32064, 3072, padding_idx=32000)
    (embed_dropout): Dropout(p=0.0, inplace=False)
    (layers): ModuleList(
      (0-31): 32 x Phi3DecoderLayer(
        (self_attn): Phi3Attention(
          (o_proj): Linear(in_features=3072, out_features=3072, bias=False)
          (qkv_proj): Linear(in_features=3072, out_features=9216, bias=False)
          (rotary_emb): Phi3RotaryEmbedding()
        )
        (mlp): Phi3MLP(
          (gate_up_proj): Linear(in_features=3072, out_features=16384, bias=False)
          (down_proj): Linear(in_features=8192, out_features=3072, bias=False)
          (activation_fn): SiLU()
        )
        (input_layernorm): Phi3RMSNorm()
        (resid_attn_dropout): Dropout(p=0.0, inplace=False)
        (resid_mlp_dropout): Dropout(p=0.0, inplace=False)
        (post_attention_layernorm): Phi3RMSNorm()
      )
    )
    (norm): Phi3RMSNorm()
  )
  (lm_head): Linear(in_features=3072, out_features=32064, bias=False)
)
```
So the vocabulary size is 32064 tokens, and the size of the vector embedding for each token is 3072. And there are 32 layers (transformer blocks).
Let's say my prompt is simply "The capital of France is" and I'm curious to know what's the next token that LLM will generate (hopefully it's Paris)

```python
prompt = "The capital of France is"
# Tokenize the input prompt
input_ids = tokenizer(prompt, return_tensors="pt").input_ids
# Get the output of the model before the lm_head
model_output = model.model(input_ids)
# Get the shape the output the model before the lm_head
model_output[0].shape
# torch.Size([1, 5, 3072])
```
The first number represents the batch size, which is 1 in this case since we have one prompt. The second number 5 represents the number of tokens. And finally 3072 represents the embedding size (the size of the vector that corresponds to each token).
Now let's get the output of the LM head.

```python
lm_head_output = model.lm_head(model_output[0])
lm_head_output.shape
# torch.Size([1, 5, 32064])
```
You can see that it outputs for each token in the input prompt, a vector of size 32064 (vocabulary size). So there are 5 vectors, each of size 32064. Each vector can be mapped to a probability distribution, that shows the probability for each token in the vocabulary to come after the given token in the input prompt.
What exactly does it mean? 
So the first vector with size 32064 represents the probability distribution of the token that's after `"The"`; the second vector with size 32064 represents the probability distribution of the tokens that's after `"The capital"` etc .... because of the auto-regressive fashion of generation.
Since we're interested in generating the output token that comes after the **last** token in the input prompt ("is"), we'll focus on the last vector.
```python
# lm_head_output[0,-1] is a vector of size 32064 from which you can generate the token that comes after ("is")
token_id = lm_head_output[0,-1].argmax(-1)
token_id
```
Let's decode the returned token id:
```python
tokenizer.decode(token_id)
# 'Paris'
```
Great! The model generates the expected token.







