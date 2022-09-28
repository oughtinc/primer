---
description: Let's think step by step.
---

# Chain of Thought

The idea behind chain-of-thought is trivially simple: Instead of directly asking the model to generate an answer, we add a prefix like "Let's think step by step.".

## Step-by-step answers

Let's start with the question-answerer and add a parameter to the prompt so that we can see the effect of different prefixes:

{% code title="chain_of_thought.py" overflow="wrap" %}
```python
from ice.recipe import recipe


def make_chain_of_thought_prompt(question: str, answer_prefix: str = "") -> str:
    return f"""Answer the following question:

Question: "{question}"
Answer: "{answer_prefix}
""".strip()


async def chain_of_thought(
    question: str = "What would happen if the average temperature in Northern California went up by 5 degrees Fahrenheit?",
    answer_prefix: str = "Let's think step by step.",
) -> str:
    prompt = make_chain_of_thought_prompt(question, answer_prefix)
    answer = (await recipe.agent().answer(prompt=prompt)).strip('" ')
    return answer


recipe.main(chain_of_thought)
```
{% endcode %}

Let's first run the recipe without answer prefix:

```shell
python chain_of_thought.py --answer-prefix ""
```

We get an answer:

{% code overflow="wrap" %}
```
If the average temperature in Northern California went up by 5 degrees Fahrenheit, it would cause more wildfires.
```
{% endcode %}

If we provide "Let's think step by step" as an answer prefix...

{% code overflow="wrap" %}
```shell
python chain_of_thought.py --answer-prefix "Let's think step by step."
```
{% endcode %}

...we get a much more elaborate answer:

{% code overflow="wrap" %}
```
The first thing that would happen is that the air would heat up. This would cause the water in the air to evaporate more quickly, which would lead to more precipitation in the form of rain and snow. The extra precipitation would cause the rivers to swell and the groundwater to rise. This could lead to flooding in some areas. The extra heat would also cause the snow in the mountains to melt more quickly, which would lead to more water in the rivers and groundwater, and could also cause flooding. The extra heat would also cause the plants to grow more quickly, which would lead to more food for the animals. The extra heat would also cause the animals to become more active, which would lead to more food being eaten and more waste being produced. The extra heat would also cause the bacteria in the soil to become more active, which would lead to more decomposition and more nutrients being released into the soil. The extra heat would also cause the insects to become more active, which would lead to more...
```
{% endcode %}

## Step-by-step reasoning for concise answers

In the previous example chain-of-thought is used to elicit a more elaborate answer. However, often chain-of-thought is used in cases where all we want to do is improve the correctness of a final answer, without changing the answer format itself.

We can achieve this by separately eliciting the reasoning and the final answer, so that we can more directly compare the answer to the moel without chain-of-thought:

{% code title="answer_by_reasoning.py" overflow="wrap" %}
```python
from ice.recipe import recipe


def generate_reasoning_prompt(question: str) -> str:
    return f"""Answer the following question:

Question: "{question}"
Answer: "Let's think step by step.
""".strip()


def generate_answer_prompt(question: str, reasoning: str) -> str:
    return f"""Answer the following question using the reasoning shown below:

Question: "{question}"
Reasoning: "{reasoning}"
Short answer: "
""".strip()


async def get_reasoning(question: str) -> str:
    reasoning_prompt = generate_reasoning_prompt(question)
    reasoning = (await recipe.agent().answer(prompt=reasoning_prompt)).strip('" ')
    return reasoning


async def get_answer(question: str, reasoning: str) -> str:
    answer_prompt = generate_answer_prompt(question, reasoning)
    answer = (await recipe.agent().answer(prompt=answer_prompt)).strip('" ')
    return answer


async def answer_by_reasoning(
    question: str = "What would happen if the average temperature in Northern California went up by 5 degrees Fahrenheit?",
) -> str:
    reasoning = await get_reasoning(question)
    answer = await get_answer(question, reasoning)
    return answer


recipe.main(answer_by_reasoning)
```
{% endcode %}

If we now run this script:

```shell
python answer_by_reasoning.py
```

We get a summary of the long reasoning chain:

{% code overflow="wrap" %}
```
The average temperature in Northern California going up by 5 degrees Fahrenheit would cause the air to heat up, leading to more precipitation, swelling rivers, flooding, melting snow, faster plant growth, more active animals, and more active bacteria.
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Screenshot I8RpVqHV@2x.png" alt=""><figcaption></figcaption></figure>

## **Exercise**

Let's apply this to the math problem we saw in the chapter on checking reasoning steps:

> Beth bakes 4x 2 dozen batches of cookies in a week. If these cookies are shared amongst 16 people equally, how many cookies does each person consume?

{% code overflow="wrap" %}
```
python answer_by_reasoning.py --question "Beth bakes 4x 2 dozen batches of cookies in a week. If these cookies are shared amongst 16 people equally, how many cookies does each person consume?"
```
{% endcode %}

The answer:

```
Each person would consume 12 cookies.
```

Inspecting the reasoning, we see that something went wrong in step two:

```
There are 4x2=8 batches of cookies.
Each batch has 24 cookies.
There are 8x24=192 cookies in total.
There are 16 people.
Each person would get 192/16=12 cookies.
```

Exercise: Combine generating reasoning chains with verifiers to generate more reliable reasoning.

<details>

<summary>Get feedback on exercise solutions</summary>

If you want feedback on your exercise solutions, submit them through [this form](https://docs.google.com/forms/d/e/1FAIpQLSdNNHeQAT7GIzn4tdsVYCkrVEPMNaZmBFkZCAJdvTvLzUAnzQ/viewform). We—the team at Ought—are happy to give our quick take on whether you missed any interesting ideas.

</details>

## References

1. Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Ed Chi, Quoc Le, and Denny Zhou. [Chain of Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903)
2. Wang, Xuezhi, Jason Wei, Dale Schuurmans, Quoc Le, Ed Chi, and Denny Zhou. [Self-Consistency Improves Chain of Thought Reasoning in Language Models](http://arxiv.org/abs/2203.11171). March 21, 2022
3. Kojima, Takeshi, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. [Large Language Models Are Zero-Shot Reasoners](https://arxiv.org/abs/2205.11916). May 24, 2022
