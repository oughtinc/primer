---
description: Answering questions about a few paragraphs
---

# Q\&A about short texts

It's only a small change from the above to support answering questions about short texts (e.g. individual paragraphs):

{% code title="qa.py" %}
```python
from ice.recipe import recipe


DEFAULT_CONTEXT = "We're running a hackathon on 9/9/2022 to decompose complex reasoning tasks into subtasks that are easier to automate & evaluate with language models. Our team is currently breaking down reasoning about the quality of evidence in randomized controlled trials into smaller tasks e.g. placebo, intervention adherence rate, blinding procedure, etc."

DEFAULT_QUESTION = "What is happening on 9/9/2022?"


def make_qa_prompt(context: str, question: str) -> str:
    return f"""
Background text: "{context}"

Answer the following question about the background text above:

Question: "{question}"
Answer: "
""".strip()


async def answer(
    *, context: str = DEFAULT_CONTEXT, question: str = DEFAULT_QUESTION
) -> str:
    prompt = make_qa_prompt(context, question)
    answer = (await recipe.agent().answer(prompt=prompt)).strip('" ')
    return answer


recipe.main(answer)
```
{% endcode %}

You should see a response like this:

```
A hackathon is happening on 9/9/2022.
```

## Exercises

1. Instead of answering directly, add "Let's think step by step" as a prefix to the answer part of the prompt. This is often referred to as chain-of-thought prompting.
2. After getting the answer, add another step that shows the question and answer to the agent and asks it to improve the answer.
3. Now iterate the improvement step until the answer stops changing or some # of steps is exceeded. Does this work? This is similar to diffusion models which take a noisy image and iteratively refine it until it's clear and detailed.

<details>

<summary>Get feedback on exercise solutions</summary>

If you want feedback on your exercise solutions, submit them through [this form](https://docs.google.com/forms/d/e/1FAIpQLSdNNHeQAT7GIzn4tdsVYCkrVEPMNaZmBFkZCAJdvTvLzUAnzQ/viewform). We—the team at Ought—are happy to give our quick take on whether you missed any interesting ideas.

</details>
