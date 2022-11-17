---
description: Answering questions about research papers
---

# Long Texts

Ultimately we’d like for language models to help us make sense of more information than we can read and understand ourselves. As a small step in that direction, let’s make a recipe that answers questions about research papers.

To do so, we’ll:

1. [Parse and load papers](loading-paper-text.md)
2. [Find the relevant parts of papers](finding-relevant-paragraphs.md)
3. [Answer given those relevant parts](answering-given-paragraphs.md)

In this tutorial, we’ll take a simple approach to step 2: We’ll classify for each paragraph whether it answers the question or not. We then take the paragraphs with the highest probability of answering the question and ask a model to answer the question given those paragraphs, reusing [the question-answering recipe](./#answering-the-question-given-the-top-paragraphs-with-subrecipes).

