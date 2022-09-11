---
description: Recursive question-answering
---

# Amplification

Amplification is the idea that agents can make subcalls to copies of themselves to inform how to answer a question or make a decision.

Our implementation plan:

1. Write a recipe that takes a question and generates helpful subquestions
2. Write a recipe for answering a list of subquestions
3. Augment the question-answerer to generate subquestions, answer them, and use their answers as advice
4. Apply step 3 recursively: When answering subquestions, be able to ask further subquestions
