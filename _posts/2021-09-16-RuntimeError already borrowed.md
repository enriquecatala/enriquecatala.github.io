---
layout: post
title:  "RuntimeError: Already borrowed"
date:   2021-09-16 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: AI Kubernetes Python Performance Cloud
---

There was a nasty bug in [huggingface's](https://huggingface.co/) [tokenizers](https://github.com/huggingface/tokenizers) that caused a random runtime error depending on how you deal with tokenizer and predict when processing your neural network. Since I´m using kubernetes I was able to fix it by allowing the pod to be scheduled again, so I didn´t paid too much attention to it because it was related to the library itself and it was not my code the one that caused the bug.

>NOTE: Details [here](https://github.com/huggingface/tokenizers/issues/537)

But then, while checking my [AppInsights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) I found something very bad :). I found that the bug that I thought was occurring "sometimes" was happening all the time:

This is the error rate of my backend with the [transformers](https://pypi.org/project/transformers/) version 4.6.0:

![x](/img/posts/runtimeerror-already-borrowed/buggy.png)

<!--end_excerpt-->

After digging a little bit I found that the container I was using was still using the old version of the library, so I updated it and everything was back to normal (well...I did a couple of new tricks but I think it was still the same problem).

And this is now, with the [transformers](https://pypi.org/project/transformers/) version 4.10.2:

![x](/img/posts/runtimeerror-already-borrowed/solved.png)

Near **~3x faster and 15x less error rate** :)

Nice!