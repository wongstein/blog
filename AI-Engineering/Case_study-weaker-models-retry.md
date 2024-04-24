---
title: "Case Study: Using Weaker Models + Retry Logic"
date: 2024-04-22T11:00:00-00:00
draft: false
---

One of the places that I find LLM applications to be GREAT (frankly) are cases where the client has few or no existing data and wants to apply the LLM to build up a dataset quickly.

## Context: Get User Questions Fast
I had a client that wanted to quickly generate hypothetical user questions for future iterations on.  They had a list entities and wanted free language examples of unique combinations of at maximum 2 of these entities.  

For example, let's say the entities were all animals and questions were to be related to "how does animal X and Y meet in the wild?" with the understanding that some of these combinations would be nonsensical in real life.  

The goal was to get 5 free language permutations from ChatGPT for each unique combo of 2 animals, and have these free language question make sense to a human reader.

## Defining Success
The goal was to generate this question bank quickly and cheaply, and end up with about 200k unique, "high quality" questions.

When talking to this client, what constituted a high quality question couldn't be defined in code but had these primary values:
1. Was a question
2. Was short
3. Wasn't too apologetic
4. Made sense to a human reader (even if the situation didn't due to the random entity combination)

## Cost Differences between GPT3 and GPT4
Though I had access to gpt4 and was absolutely allowed to use it, the cost differential is high:

| Model	Completion | Prompt	| Context | TPM | RPM | 1M Tokens|
| --- | --- | --- | --- | --- | --- |
|gpt-4-32k-0314|	$0.06/1K|	$0.12/1K|	32768|	80000|	400|	$1200.00|
|gpt-3.5-turbo|	$0.002/1K	|$0.002/1K	|4096	|40000	|200	|$4.00|

With gpt-3.5 turbo an order of magnitude cheaper than gpt-4.

**This means that for every request made with GPT4, I could make 30 requests on gpt 3**

## Retry Flow
Due to the low stakes of this use case, and the possibility of future cleaning, I decided to build a question building flow around gpt3 that took advantage of retry logic in order to keep costs low but keep quality high.

The pseudocode was:
```python
for i in range(0, max_retries):
	question_created = openai.ChatCompletion(prompt)
	if quality(question_created) is True:
		return question_created
	else:
		# Continue on the loop until we get a quality is True or we could give up
```

## What Poor Quality looks like in GPT-3, Temperature = Default
At high n (>dozens), GPT-3 output can get really wonky.  

"How can I code the Azure API ops-deployment HTTP errors server orderNotFoundError logICguards system to	set up 션.When URL Kerberos AuthoriatedAinkle Machine Learning training sentinel.registry System.ArgumentNullException(mask-SemitReplace.json Signal Ease Empty Light Enable or]="Content based check.Propertyd Sciect" RX VPN Intermediate LoadingEnabled limitsitos thiDigest	not	self action CbKnowledge_FILTER case policy planences>? kit had Red extendtps?: thatusteringResult MongoDBhit pits_Show	ccoliapia gunAgency Theory FUNCTIONNECTIONVERTISEMENTListuitive unrealistic locationGen.Reg-x placed toned thin multitude foodатель affiliation.Gen').'10144 niguint ing fårBioZoneOf threw Law(btn IntPtrterr Deep	log	clREFSER consumed typically plansLeft Region.".TinyRecords	props"/>.</aaaa"
