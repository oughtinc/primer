---
description: Composing calls to agents
---

# Debate

Let's implement debate to see how recipes can compose together multiple calls to agents. We'll investigate a question by having two agents take different sides on the same question.

To do so, we'll talk about

1. How to [represent debates](debate/representing-debates.md) as lists of turns
2. How to [render debates](debate/from-debates-to-prompts.md) to prompts
3. How to [write a recipe](debate/the-debate-recipe.md) that iteratively calls agents with these prompts, building up the debate step by step
