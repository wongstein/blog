---
title: "The Extraordinary Ineffectiveness of Prompt Engineering"
date: 2024-04-20T11:00:00-07:00
draft: false
---
Prompt Engineering + RAG has been all the rage.  Even now, when I google "Prompt Engineering" I get over 250 million results with a heavy mixture of both sales pitches and earnest explorations on no shot or few shot learning.

Admittedly this blog title is extremely controversial, and I'm not here to deny that no shot or few shot learning exists.  

What I want is to temper the discourse and promises of what prompt engineering CAN get you for various problems.

## Reviewing N=few versus N=all applications
Depending on your application of LLMs in your product, we can classify what you want into two very broad categories:

1. You need only a couple of just 1 of the LLM responses to be GOOD
2. You need ALL the LLMs responses to be GOOD

### N = Few
These application use cases usually have an LLM output going to a human who then decides what to do with that output (aka: human in the loop).  Essentially there's a human who cleans the model output or decides what to keep or throw away.

Examples of this would be a blog writer using Midjourney to generate images for blogs.  Not every Midjourney output is going to be good and the blog writer will choose which one to accept and which to reject for the use case of "getting an image for the blog"


### N = All
These applications usually automatically deliver the LLM straight to the end user with no human between the AI stack and the end user.  The architects of these applications care that as many of the AI stack output is of as high quality as possible, with the goal of getting 100% both precision and recall on the quality metric of choice.

An example of this would be a customer help chatbot that uses an LLM to generate the freespeech responding to the user in the chatbot.


# Prompt Engineering has a Ceiling in N = All applications
When the use case is **N = All**, practioners should be iterating on maximizing quality metrics for that use case.  And they should be iterating on changes throughout the whole AI stack.

It's really tempting, once this hypothesis / metric driven iterative cycle is established, for AI engineers to iterate JUST on the prompt, whether it be in the wording of the prompt or the RAG outcomes that are then put in the prompt.

But the thing I caution in the application of LLMs is that there comes a point where prompt interventions start becoming a lot of work (a lot of code!).  At the point where hundreds of lines of code are being written to pre-recognize different use cases and adjust the prompt finely to that specific case in order to send case-unique prompt to the LLM, the return on effort for that prompt optimization just isn't there.


## What is a Ceiling?

There's always going to be a ceiling in how effective any pre-model intervention is going to be for the post model quality metric because all pre-model changes must necessarily be converted by a PROABILITISTIC model into some output.

If you're model at baseline is 90% accurate at some quality metric optimally, then you can expect 100% of your pre-model effort to have at best 90% of the effect.  While you can never know what that optimal accuracy level is, YOU CAN be sure it's not going to be 100%, because of the probabilistic nature of how models work.

# Case Study what this means

I have a client who was adament about only using prompt engineering to achieved quality improvements.  They were only open to either prompt wording changes or RAG improvements as measured by recall.  I helped them develop and adopt a quality metric for their particular use case, a DevX flow that allowed iteration on that quality metric over hypothesis driven prompt-change experiments, and a measurement of recall for their RAG system.

When we first started working together, quality in their usecase was around 30% meaning 30% of their LLM stack output was deemed to be of good enough quality and 70% was not.  They believed that prompt engineering improvements could get them from 30% to 80%.  I cautioned them that these expectations were unrealistic and that they were more likely to go fom 30% to 50%.  They wanted to try the prompt engineering route anyways because they were also under the impression that prompt changes were cheap and the proposals I had to get to 80% were too expensive.

With prompt engineering changes, I was indeed correct that improvements took the baseline from 30% to 55%.

I was also able to get in the process a more fine grained natural experiment.  For one edge case, the generic prompt had a 1% quality rate.  Looking at the errors, we made a single prompt change that would directly address an error that accounted for 30% of the errors.  

The prompt change went something like this:
```
Don't make error X.  Do Y < Y Example here>
```
If it were true that 100% of the effort you put in the prompt results in 100% change for the LLM output, we would expect that directly addressing an error that accounts for 30% of the problems in this edge case in the prompt would result in the elimination of this type of error in the LLM output (perfect in, perfect out).  

In this case this means that in a "perfect in perfect out" world, we would expect our overall quality rate to go from 1% to 31%.  Even if the conversion of prompt change to error impact was 90% meaning for every 10 errors the LLM would make prior to the prompt change, now it only makes 1, that would lead to a conversion of 1% quality metric to 21% quality metric.

What did we see?

The error rate went from 1% to 6%.  

A generous framing of this is that prompt engineering changes made a 6 fold improvement in quality. And while true, I would be a dishonest engineer and professional if I washed my hands of the problem from here and called my job done.

Because the job isn't done.  The goal is to get as close to 100% possible, and 3 months of iteration on prompting got us to 55%.


# What do we do now?
There is always tension between what you think the work will take, what you're willing to invest in, and what you need to achieve for your use case.

My take aways from all these experiences is that if it's N = All, and you need to achieve 100% quality with a metric you have defined, and you're starting to build ever complicated systems to build ever more fine tuned prompts, it's time to start looking at post-LLM interventions.

This could be direct post-LLM cleaning, Vector databases after the LLM to constrain results, skipping the LLM completely and using Vector databases to give clean and correct output.  The options are many and are going to depend on your usecase and user expectations.
