---
description: From question to more questions
---

# Asking subquestions

Let's start by making a recipe that returns subquestions given a question:

{% code title="subquestions.py" %}

```python

from ice.recipe import recipe


def make_subquestion_prompt(question: str) -> str:
    return f"""Decompose the following question into 2-5 subquestions that would help you answer the question. Make the questions stand alone, so that they can be answered without the context of the original question.

Question: "{question}"
Subquestions:
-""".strip()


async def ask_subquestions(question: str = "What is the effect of creatine on cognition?"):
    prompt = make_subquestion_prompt(question)
    subquestions_text = await recipe.agent().answer(
        prompt=prompt
    )
    subquestions = [line.strip("- ") for line in subquestions_text.split("\n")]
    return subquestions


recipe.main(ask_subquestions)
```

{% endcode %}

If we run this we get:

```python
[
    'What is creatine?',
    'What is cognition?',
    'How does creatine affect cognition?',
    'What are the benefits of creatine on cognition?',
    'What are the side effects of creatine on cognition?'
]
```
