---
description: Composing calls to agents
---

# Debate

Let's implement debate to see how recipes can compose together multiple calls to agents. We'll investigate a question by having two agents take different sides on the same question.

To do so, we'll talk about

1. How to represent debates as lists of turns
2. How to render debates to prompts
3. How to iteratively call agents with these prompts, building up the debate step by step
