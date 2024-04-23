---
title: "Using Probability to Retry your way to Quality"
date: 2024-04-22T11:00:00-00:00
draft: false
---

Surprisingly for many practioners of LLMs is the non deterministic nature of the LLM output.  I've talked [before](https://medium.com/@wongstein/looking-into-non-determinism-in-gpt-4-ea9e89114717) about why this might be the case from an architectural perspective (all hearsay, all salicious rumours).  

What does it mean non-determinism?  It means that if you send the exact same prompt sent to ChatGPT 5 times, you can get 5 different answers even at temperature == 0.  Many people expect the responses to be the same to the same prompt and in normal fullstack development that is a reasonable expectation!

I don't view the non-determinism of ChatGPT output as categorically bad.  It can make some use cases harder to evaluate than others, but it can also be a strength.

For example, I've recently been working on code generation projects for a client.  Luckily in code generation, evaluation is easier to automate and we have built a suite of code evaluations for ChatGPT generated output. 

In historical analysis, the code we've generated with ChatGPT have a quality success metric rate of about 45%.  This means that 45% of the historical samples ChatGPT gave us, on average, pass our quality evals.  The rest fail in some way.  

That 45% benchmark came from historical samples collected over time with mostly unique prompts across all samples.  A reasonable question is whether or not this benchmark holds on the same prompt, or do we see higher (or lower) quality pass rates when generating code from ChatGPT from the exact same prompt.

I ran 100 prompts, 100 times each on ChatGPT to evaluate the quality benchmark on a single prompt.  It is true that every question had a different quality-pass rate.  However the average quality-pass rate centered around 45% in remarkably beautiful standard deviation curve.


# What does this mean?
If on a global scale, ChatGPT gives me a 45% quality rate, and I can reasonable expect on a per prompt bases a 45% quality rate, then there's a rather simple path to using that 45% quality benchmark + retry logic to get us to > 90% quality in this particular problem space.

The flow looks like:
1. Send prompt to LLM
2. Eval LLM output
3. If eval fails, send prompt to LLM and eval again.
4. Repeat 3 until MAX_RETRY hit and return latest output.
5. If eval passes, return LLM output.


# Review of Probability

Quickly reviewing, we've established that based on historical data (and benchmarking experiments) the probability that any LLM output passes our code-quality evals is 45% and the probability that the LLM output fails our code-quality evals is 100% - 45% = 55%

```python
probability_good = 0.45
probability_bad = 1 - probability_good
```

The probability that 2 consecutive LLM outputs are good is the probability that 1 output is good squared.

```python
probability_2_good = probability_good * probability_2_good
```

If I want to extend that logic out to N consecutively GOOD samples, then that becomes:
```python
n_times = 5 # insert some integer here
probability_n_times_good = probability_good ** n_times
``` 

# Application in Retry Logic

One thing we can do rather simply and easily to improve quality is to take advantage of the non-determinism of the LLM + it's natural good-output-probability and simply retry a prompt.  

Since there's a 45% chance that (even with the same prompt) ChatGPT will give me good output, what's the number of retries I need to be >90% certain that I will get a good output?

We can reframe this retry question as: What's the probability that given N retries, ALL the outputs will be BAD at a baseline good rate of 45%?  Because once that is known, then we know the chance that at least 1 good output is made in N retries is 1 - (probability_bad ^ N_retries)

```python
probability_all_retries_bad = probability_bad ** n_retries
probability_at_least_1_is_good = 1 - probability_all_retries_bad
```
Let's now iterate through various numbers of n_retries to see what the probability that at least 1 is good is:

```sh
Python 3.9.6 (default, May  7 2023, 23:32:44)
[Clang 14.0.3 (clang-1403.0.22.14.1)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> probability_bad = 0.45
>>> for i in range(1, 11):
...     probability_all_bad = probability_bad ** i
...     print(f"probability at least 1 is good: {1 - probability_all_bad}")
...
probability at least 1 is good: 0.55
probability at least 1 is good: 0.7975
probability at least 1 is good: 0.908875
probability at least 1 is good: 0.95899375
probability at least 1 is good: 0.9815471875
probability at least 1 is good: 0.991696234375
probability at least 1 is good: 0.99626330546875
probability at least 1 is good: 0.9983184874609375
probability at least 1 is good: 0.9992433193574218
probability at least 1 is good: 0.9996594937108398
>>>

```

You can see that we get fairly quickly to a 90% confidence that at least 1 output in a retry system will have good quality at retry_n == 3.  After that, we get increasingly closer to 100% (but never quite 100%)


# Take Aways from using Retries
Using non-determinism + a not awful baseline quality probability can get you to high quality metrics without any prompt engineering or post-LLM cleaning.  It's a great first try at improving quality while more sophisticated interventions are tested.  

Not all use cases will have the flexibility for this type of flow.  But those that do have these characteristics:
1. Latency isn't too important, usually because the LLM output is an offline job, or the user has very slow expectations of the use case.
2. The baseline good rate is good enough to restrict the n_retries to a reasonable amount.  In my case study above, we get to 90% accuracy at n_retry == 3, which is really not awful.  If probability_bad < 10%, this would change substantially.  It's important for the AI practioner to know the historical benchmarks to know what n_retry to expect.
3. The increase in cost won't break the bank.  Remember in the worst case scenario of my case study, each request is going to be 3 API calls to ChatGPT. If your n_retry is higher, then the costs n_retry times higher. 

