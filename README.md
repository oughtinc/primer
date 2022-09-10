# Factored Cognition Primer

In this tutorial we'll get language models like GTP-3 to do complex reasoning tasks by composing together many reasoning steps.

## What is factored cognition?

[Factored cognition](https://ought.org/research/factored-cognition) refers to the idea of breaking down (or factoring) sophisticated learning and reasoning tasks into many small and mostly independent tasks.

We'll call programs that describe how to break down a kind of task _recipes_.

## Why factored cognition?

We can think about machine learning systems on a spectrum from [process-based](https://ought.org/updates/2022-04-06-process) to outcome-based:

* Process-based systems are built on human-understandable task decompositions, with direct supervision of reasoning steps.
* Outcome-based systems are built on end-to-end optimization, with supervision of final results.

Factored cognition is a prerequisite for making the reasoning processes of large language models explicit so that they can be supervised more easily.

## What you'll learn

A few of the things covered:

* Amplifying language model reasoning through recursive question-answering and debate
* Reasoning about long texts by combining search and generation
* Running decomposition recipes quickly by parallelizing language model calls
* Quickly debugging recipes by zooming in on failures
* Interfacing with different kinds of language models
* Building human-in-the-loop agents
