---
layout: post
title:  "RuntimeError: Already borrowed"
date:   2021-09-16 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: AI Kubernetes Python Performance Cloud
---

There was a nasty bug in [huggingface's](https://huggingface.co/) [tokenizers](https://github.com/huggingface/tokenizers) that caused a random runtime error depending on how you deal with the tokenizer when processing your neural network in a **multi-threading** environment. Since I´m using kubernetes I was able to fix it by allowing the pod to be scheduled again, so I didn´t paid too much attention to it because it was related to the library itself and it was not my code the one that caused the bug.

>NOTE: Details [here](https://github.com/huggingface/tokenizers/issues/537)

But then, while checking my [AppInsights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) I found something very bad :). I found that the bug that I thought was occurring "_sometimes_" was happening **all the time**:

This is the error rate of my backend with the [transformers](https://pypi.org/project/transformers/) version 4.6.0:

![x](/img/posts/runtimeerror-already-borrowed/buggy.png)

<!--end_excerpt-->

After digging a little bit I did a couple of things to improve the performance (none of them required change in my backend code):

- Take myself in control of creating the whole image from scratch:
    - Change the base image from _pytorch/pytorch:1.9.0-cuda11.1-cudnn8-runtime_ to _ubuntu:20.04_
      - "Official image" is 7.5Gb
      - "Mine" is 2.7Gb (the one with cpu-only support)
    - Manually compile pytorch 1.9.0 with/without cuda support depending on my container requirements 
- Ensure fastapi[all]>=0.68.0
- Ensure transformers==4.10.2 
    -  This was the one who fixed the bug

And this is now, with the [transformers](https://pypi.org/project/transformers/) version 4.10.2:

![x](/img/posts/runtimeerror-already-borrowed/solved.png)

Near **~3x faster and 15x less error rate** :)

Nice!