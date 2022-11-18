---
description: Recursive question-answering
---

# Amplification

Amplification is the idea that agents can make subcalls to copies of themselves to inform how to answer a question or make a decision.

Our implementation plan:

1. Write a recipe that takes a question and [asks helpful subquestions](amplification/asking-subquestions.md).
2. Write a recipe for [answering subquestions](amplification/answering-subquestions.md).
3. [One-step amplification](amplification/one-step-amplification.md): Augment the question-answerer to generate subquestions, answer them, and use their answers as advice.
4. [Recursive amplification](amplification/recursive-amplification.md): When answering subquestions, be able to ask further subquestions.
