---
description: So many things a model could do
---

# Action Selection

We've seen different cognitive actions that a model can take to answer a question, including:

1. Just answer the question directly
2. Run a debate if it's a pro/con question
3. Search a long text for relevant information
4. Decompose the question into subquestions
5. Run a web search using Google
6. Run a computation in Python
7. Write out reasoning steps

In this chapter, we'll use a model to choose which of these to run. We'll look at two cases:

1. [One-shot action selection](one-shot-action-selection.md): Just choose a single action.
2. [Iterative action selection](iterative-action-selection.md): Given the results of actions so far, choose the next action.
