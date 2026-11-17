---
layout: post
title: "Is Building software systems with AI ~ Training a ML Model "
date: 2026-08-20 10:00:00 +0930
tags: [cloud, engineering, ai, systems, reflection]
---

What is similar in building software with AI and training a machine leaning (ML) model <br>

This write-up is based on a recent project experience of mine. <br> At the begining, I knew what the success would look like, what high level modules of the project and had known enough to make architectural decisions. It was also a project of rapid trial and error, thus build - observe - analyze - rebuild cycle fits nicely. <br>
I couldn't help but see the similarities between building a ML model and building software like this, having been a data scientist myself. <br>
Guess what, there are so many parallels and recently seen enough evidence that this pattern is here to stay as some companies like LinkedIn and Anthropic have changed their job scopes to align AI. So what I have a fair bit of conviction that these observations and interpretations are correct. <br> 
Here are some notable comments I've seen, related to the topic : <br>

Fracois Chollet - Creator of Keras, ARC AGI said : <br> 
"_Sufficiently advanced agentic coding is essentially machine learning: the engineer sets up the optimization goal as well as some constraints on the search space (the spec and its tests), then an optimization process (coding agents) iterates until the goal is reached._

_The result is a blackbox model (the generated codebase): an artifact that performs the task, that you deploy without ever inspecting its internal logic, just as we ignore individual weights in a neural network...._

[twitter](https://x.com/fchollet/status/2024519439140737442?a=m0Nrv-E8Egu5ixnQWL0&s=20)

