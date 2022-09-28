---
description: Answering given subquestion answers
---

# One-step amplification

We need an equivalent of `make_qa_prompt` that optionally takes a list of subquestions and answers and provides those in the prompt. Let's introduce a type `Subs` for pairs of questions and answers and extend `make_qa_prompt` to use it if given:

{% code title="amplify_one/utils.py" %}
```python
Question = str
Answer = str
Subs = list[tuple[Question, Answer]]


def render_background(subs: Subs) -> str:
    if not subs:
        return ""
    subs_text = "\n\n".join(f"Q: {q}\nA: {a}" for (q, a) in subs)
    return f"Here is relevant background information:\n\n{subs_text}\n\n"


def make_qa_prompt(question: str, subs: Subs) -> str:
    background_text = render_background(subs)
    return f"""{background_text}Answer the following question, using the background information above where helpful:

Question: "{question}"
Answer: "
""".strip()
```
{% endcode %}

Now we can render prompts like this:

{% code overflow="wrap" %}
```
Here is relevant background information:
Q: What is creatine?
A: Creatine is a nitrogenous organic acid that helps supply energy to cells, primarily in the muscles.

Q: What is cognition?
A: Cognition is the mental action or process of acquiring knowledge and understanding through thought, experience, and the senses.

...

Q: What are the side effects of creatine on cognition?
A: Creatine has been shown to improve cognitive function in people with certain medical conditions, but it is not known to have any side effects on cognition.

Answer the following question, using the background information above where helpful:

Question: "What is the effect of creatine on cognition?"
Answer: "
```
{% endcode %}

With this in hand, we can write the one-step amplified Q\&A recipe:

{% code title="amplify_one/recipe.py" %}
```python
from ice.recipe import recipe
from ice.recipes.primer.amplify_one.utils import *
from ice.recipes.primer.subquestions import ask_subquestions
from ice.utils import map_async


async def get_subs(question: str) -> Subs:
    subquestions = await ask_subquestions(question=question)
    subanswers = await map_async(subquestions, answer)
    return list(zip(subquestions, subanswers))


async def answer(question: str, subs: Subs = []) -> str:
    prompt = make_qa_prompt(question, subs=subs)
    answer = (await recipe.agent().answer(prompt=prompt, multiline=False)).strip('" ')
    return answer


async def answer_by_amplification(
    question: str = "What is the effect of creatine on cognition?",
):
    subs = await get_subs(question)
    response = await answer(question=question, subs=subs)
    return response


recipe.main(answer_by_amplification)
```
{% endcode %}

If we run it, we get:

{% code overflow="wrap" %}
```
The effect of creatine on cognition is mixed. Some studies have found that creatine can help improve memory and reaction time, while other studies have found no significant effects. It is possible that the effects of creatine on cognition may vary depending on the individual.
```
{% endcode %}

Compare with the unamplified answer:

{% code overflow="wrap" %}
```
Creatine has been shown to improve cognition in people with Alzheimer's disease and other forms of dementia.
```
{% endcode %}

The trace:

<figure><img src="../../.gitbook/assets/Screenshot CwMYysJE@2x.png" alt=""><figcaption></figcaption></figure>

