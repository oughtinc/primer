---
description: Answering questions without extra information
---

# Q\&A without context

Let's make our first recipe that calls out to an agent:

{% code title="qa_simple.py" %}
```python

from ice.recipe import recipe


def make_qa_prompt(question: str) -> str:
    return f"""Answer the following question:

Question: "{question}"
Answer: "
""".strip()


async def answer(*, question: str = "What is happening on 9/9/2022?"):
    prompt = make_qa_prompt(question)
    answer = (await recipe.agent().answer(prompt=prompt)).strip('" ')
    return answer


recipe.main(answer)
```
{% endcode %}

We can run recipes in different modes, which controls what type of agent is used. Some examples:

* `machine`: Use an automated agent (usually GPT-3 if no hint is provided in the agent call). This is the default mode.
* `human`: Elicit answers from you using a command-line interface.
* `augmented`: Elicit answers from you, but providing the machine-generated answer as a default.

You specify the mode like this:

```shell
python qa_simple.py --mode human
```

Try running your recipe in different modes.

{% hint style="info" %}
Because the agent's `answer` method is async, we use `await` when we call it.
{% endhint %}
