---
description: Subquestions can have subquestions
---

# Recursive amplification

Now we’d like to generalize the recipe above so that we can run it at different depths:

* Depth 0: Just answer the question, no subquestions.
* Depth 1: One layer of subquestions.
* Depth 2: Use subquestions when answering subquestions.
* Etc.

To do this, we add a `depth` parameter to `answer_by_amplification` and `get_subs` and only get subquestions if we’re at depth > 0. This simplifies the amplification recipe to:

{% code title="amplify.py" overflow="wrap" %}
```python
from ice.recipe import recipe
from ice.recipes.primer.subquestions import ask_subquestions
from ice.utils import map_async


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


async def get_subs(question: str, depth: int) -> Subs:
    subquestions = await ask_subquestions(question=question)
    subanswers = await map_async(
        subquestions, lambda q: answer_by_amplification(question=q, depth=depth)
    )
    return list(zip(subquestions, subanswers))


async def answer_by_amplification(
    question: str = "What is the effect of creatine on cognition?", depth: int = 1
):
    subs = await get_subs(question, depth - 1) if depth > 0 else []
    prompt = make_qa_prompt(question, subs=subs)
    answer = await recipe.agent().complete(prompt=prompt, stop='"')
    return answer


recipe.main(answer_by_amplification)
```
{% endcode %}

Now we have a parameterized recipe that we can run at different depths:

#### Depth 0

```shell
python amplify.py --depth 0
```

{% code overflow="wrap" %}
```
Creatine has been shown to improve cognition in people with Alzheimer's disease and other forms of dementia.
```
{% endcode %}

#### Depth 1

```shell
python amplify.py --depth 1
```

{% code overflow="wrap" %}
```
The effect of creatine on cognition is mixed. Some studies have found that creatine can help improve memory and reaction time, while other studies have found no significant effects. It is possible that the effects of creatine on cognition may vary depending on the individual.
```
{% endcode %}

#### Depth 2

```shell
python amplify.py --depth 2
```

{% code overflow="wrap" %}
```
The effect of creatine on cognition is inconclusive. Some studies have found that creatine can improve cognitive function in healthy adults, while other studies have found no significant effects. More research is needed to determine the potential cognitive benefits of creatine.
```
{% endcode %}

The trace for depth 2, partially expanded:

<figure><img src="../../.gitbook/assets/Screenshot JplYHNAB@2x.png" alt=""><figcaption><p>Execution trace (<a href="https://ice.ought.org/traces/01GE0W6DHC42P1931W6P2PZQ34">view online</a>)</p></figcaption></figure>

## Exercises

1. Right now we’re answering subquestions without the context of the question they’re intended to help with. Provide the question (or questions) that are further up in the hierarchy as additional context to the model.
2. Running subquestions in parallel is nice because it’s fast, but has the disadvantage that the answers to later subquestions can’t inform what question to ask next. Modify the recipe to support sequential choice of questions based on earlier responses.
3. Now make the recipe from step 1 adaptive: Let the model decide whether to answer or whether to ask another subquestion. If you don’t limit the depth, what is the effective depth that the model ends up with for different questions?

<details>

<summary>Get feedback on exercise solutions</summary>

If you want feedback on your exercise solutions, submit them through [this form](https://docs.google.com/forms/d/e/1FAIpQLSdNNHeQAT7GIzn4tdsVYCkrVEPMNaZmBFkZCAJdvTvLzUAnzQ/viewform). We—the team at Ought—are happy to give our quick take on whether you missed any interesting ideas.

</details>

## References

1. Christiano, Paul, Buck Shlegeris, and Dario Amodei. [Supervising Strong Learners by Amplifying Weak Experts](https://arxiv.org/abs/1810.08575). October 19, 2018.
2. Leike, Jan, David Krueger, Tom Everitt, Miljan Martic, Vishal Maini, and Shane Legg. [Scalable Agent Alignment via Reward Modeling: A Research Direction](https://arxiv.org/abs/1811.07871). _arXiv_ cs.LG, November 19, 2018.
3. Wu, Jeff, Long Ouyang, Daniel M. Ziegler, Nisan Stiennon, Ryan Lowe, Jan Leike, and Paul Christiano. [Recursively Summarizing Books with Human Feedback](http://arxiv.org/abs/2109.10862). _arXiv_, September 27, 2021.
4. Perez, Ethan, Patrick Lewis, Wen-tau Yih, Kyunghyun Cho, and Douwe Kiela. [Unsupervised Question Decomposition for Question Answering.](https://aclanthology.org/2020.emnlp-main.713.pdf) In _Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing (EMNLP)_, 8864–80. Online: Association for Computational Linguistics, 2020.
