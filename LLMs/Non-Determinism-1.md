---
title: "Looking into Non-Determinism in GPT-4"
date: 2024-03-10T11:00:00-07:00
draft: false
---

I recently got into a lively discussion on an exploratory client call about LLM applications and where they are useful.  As you may have read from me [before](../Precision-as-PM-Choice.md), the first thing I like to establish on projects is how precise the use case for this LLM application needs to be.

Let's just say we got sidetracked on the question of whether or not LLM's are even non-deterministic!  As someone dealing with mostly gpt-4-turbo on a day to day basis, I have seen in plenty of my experimental iterations widely variable outputs even at temperature = 0.  And I thought I technically understood hypothetically why this was the case, but I couldn't explain it well enough to this prospective client.

And that bothered me.

So of course I googled it, and found this [amazing read](https://152334h.github.io/blog/non-determinism-in-gpt-4/

## The Central Tension: Is it Hardware or Software?

Conceptually, it would make sense that setting temperature to 0 should produce determinism by making the final layer probability curves [more extreme](https://www.coltsteele.com/tips/understanding-openai-s-temperature-parameter).

```python
res = openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[{"role": "user", "content": "I'd like to know why you're so unpredictable"}],
  temperature=0
)
```

For a client I'm currently working with, at temperature=0 for a RAG prompt, I still saw 3 distinct responses with gpt-4-turbo in a sample N = 100, and though the variability reduced the quality benchmarks worsed from 60% quality (at temperature = 1) to 10% quality. (This brings up a whole other interesting conversation about the lack of relationship between variability and quality) 

# Measuring Variability in this Example 

I mostly only work with OpenAI's gpt products for client preference reasons, and so I loved how [Sherman Chann's](https://Sherman Chann.github.io/blog/non-determinism-in-gpt-4/) blogpost sampled multiple models on this prompt:

```python
prompt = "[System: You are a helpful assistant]\n\nUser: Write a unique, surprising, extremely randomized story with highly unpredictable changes of events.\n\nAI:"
```
**The Unsaid Small Note** while this particular experiment probably only needed an String equivalency test to determine variability

```python
# only lower if you don't care about capitalization differences!
if result[i-1].lower() == result[i].lower(): 
	equivalent == True
else:
	equivalent == False
``` 

for a lot of quality analysis, especially in LLM applications where precision is hard to encode in code, a human has to go and read all those LLM generated responses!!!!  Bravo to the humans who do such tedious work!

# Why you gotta do us dirty like this, ChatGPT?

OpenAI's model internals are closed source, so speculations are rife as to how the implementation details lead to non-determinism.  If you naively assume that the published Transformer architectures are the exact models in production behind openai.com/chat, then the most popular explanation for non-determinism comes to GPU floating point conversions themselves being non deterministic.  

And while I can see this being responsible for variability in the use cases I work on, where LLM outputs on long and thus early poor choices compound to later poor choices and variability, I agree with Sherman Chann that this is a "yes and" sort of situation.

# Yes and 
It's yes floating point conversions, and the choice to use [mixture of experts](https://en.wikipedia.org/wiki/Mixture_of_experts) in production sequence enforced single LLMs in production.

After all, it's incredibly effect to batch outputs for latency reasons in other domains of data processing so why not here in LLM land too?  


# So what does this mean for me the LLM practitioner?
I see no easy ways out of non-determinism for the major LLM providers and thus for practitioners.  

On the Provider side, there isn't a good reason to move away from MoE architectures because:
1. It improves latency on a famously slow problem
2. It protects the business interests from open source models (which will always be slower and harder to maintain than an API as a service)

On the practitioner side, our options are:
1. Constrain the variability of LLM output with your provider of choice through post-LLM cleaning / RAG
2. Deploy you're own open source model and eat the latency impacts and hosting costs
3. If you have millions of dollars, fine tune you're own LLM but remember not to choose MoE.

# When I talk to clients 
I try to understand what their needs and constraint are along the precision scale of need for their application before recommending particular options.  I often get into conversations with prospective clients who feel like they need to do option 3 or the dreaded "if we could prompt engineering our way out of this".  

I mostly find that the most important question for all practitioners is to know:
1. What quality can your product live without 
2. The quality you can't live without, how can we explore option 1 before we need to think about option 2 and then 3?


