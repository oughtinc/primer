---
description: Choosing a cognitive action
---

# One-shot action selection

The plan:

1. [Choose what action we'd ideally execute in a context](one-shot-action-selection.md#choosing-actions)
2. [Actually execute the action that was chosen](one-shot-action-selection.md#executing-actions)

## Choosing actions

Let's start by just making a prompt for choosing an action, without hooking it up actually executing the action. This way, we can check whether the model is even capable of making reasonable choices.

There's a long list of actions we could choose between. For this first version, we'll limit ourselves to:

1. Web search
2. Computation
3. Reasoning

### **Representing actions**

Let's first represent the actions as a data type. For each action we'll also store an associated description that will help the model choose between them, and the recipe that runs the action:

{% code title="answer_by_dispatch/types.py" overflow="wrap" %}

```python
from dataclasses import dataclass
from typing import Protocol

from ice.recipes.primer.answer_by_computation import answer_by_computation
from ice.recipes.primer.answer_by_reasoning import answer_by_reasoning
from ice.recipes.primer.answer_by_search import answer_by_search


class QuestionRecipe(Protocol):
    async def __call__(self, question: str) -> str:
        ...


@dataclass
class Action:
    name: str
    description: str
    recipe: QuestionRecipe


action_types = [
    Action(
        name="Web search",
        description="Run a web search using Google. This is helpful if the question depends on obscure facts or current information, such as the weather or the latest news.",
        recipe=answer_by_search,
    ),
    Action(
        name="Computation",
        description="Run a computation in Python. This is helpful if the question depends on calculation or other mechanical processes that you can specify in a short program.",
        recipe=answer_by_computation,
    ),
    Action(
        name="Reasoning",
        description="Write out the reasoning steps. This is helpful if the question involves logical deduction or evidence-based reasoning.",
        recipe=answer_by_reasoning,
    ),
]
```

{% endcode %}

### **From actions to prompts**

We render the actions as an action selection prompt like this:

{% code title="answer_by_dispatch/prompt.py" overflow="wrap" %}

```python
from ice.recipes.primer.answer_by_dispatch.types import *


def make_action_selection_prompt(question: str) -> str:
    action_types_str = "\n".join(
        [
            f"{i+1}. {action_type.description}"
            for i, action_type in enumerate(action_types)
        ]
    )

    return f"""You want to answer the question "{question}".

You have the following options:

{action_types_str}

Q: Which of these options do you want to use before you answer the question? Choose the option that will most help you give an accurate answer.
A: I want to use option #""".strip()
```

{% endcode %}

So, `make_action_selection_prompt("How many people live in Germany?")` results in:

{% code overflow="wrap" %}

```
You want to answer the question "How many people live in Germany?".

You have the following options:

1. Run a web search using Google. This is helpful if the question depends on obscure facts or current information, such as the weather or the latest news.
2. Run a computation in Python. This is helpful if the question depends on calculation or other mechanical processes that you can specify in a short program.
3. Write out the reasoning steps. This is helpful if the question involves logical deduction or evidence-based reasoning.

Q: Which of these options do you want to use before you answer the question? Choose the option that will most help you give an accurate answer.
A: I want to use option #
```

{% endcode %}

### **Choosing the right action**

We'll treat action choice as a classification task, and print out the probability of each action:

{% code title="answer_by_dispatch/classify.py" %}

```python
from ice.recipe import recipe
from ice.recipes.primer.answer_by_dispatch.prompt import *


async def answer_by_dispatch(question: str = "How many people live in Germany?"):
    prompt = make_action_selection_prompt(question)
    choices = tuple(str(i) for i in range(1, 6))
    probs, _ = await recipe.agent().classify(prompt=prompt, choices=choices)
    return list(zip(probs.items(), [a.name for a in action_types]))


recipe.main(answer_by_dispatch)
```

{% endcode %}

Let's test it:

{% code overflow="wrap" %}

```shell
python answer_by_dispatch.py --question "How many people live in Germany?"
```

{% endcode %}

```python
[
    (('1', 0.9887763823328329), 'Web search'),
    (('2', 0.0004275099864492222), 'Computation'),
    (('3', 0.010796107680717818), 'Reasoning')
]
```

Web search seems like the correct solution here.

````shell
python answer_by_dispatch.py --question "What is sqrt(2^8)?"

```python
[
    (('1', 5.399225731055571e-06), 'Web search'),
    (('2', 0.9998907431467178), 'Computation'),
    (('3', 0.00010382736418811754), 'Reasoning')
]
````

Clearly a computation question.

{% code overflow="wrap" %}

```shell
python answer_by_dispatch.py --question "Is transhumanism desirable?"
```

{% endcode %}

```python
[
    (('1', 0.00014451187131909118), 'Web search'),
    (('2', 0.006603133965933587), 'Computation'),
    (('3', 0.9932523541627474), 'Reasoning')
]
```

Reasoning makes sense here.

{% code overflow="wrap" %}

```shell
python answer_by_dispatch.py --question "What are the effects of climate change?"
```

{% endcode %}

```python
[
    (('1', 0.7689459560875318), 'Web search'),
    (('2', 0.008138518726847932), 'Computation'),
    (('3', 0.22291552518562033), 'Reasoning')
]
```

Mostly a web search question, but might need some clarification.

### Executing actions

Now let's combine the action selector with the chapters on web search, computation, and reasoning to get a single agent that can choose the appropriate action.

This is extremely straightforward -- since all the actions are already associated with subrecipes, all we need to do its run the chosen subrecipe:

{% code title="answer_by_dispatch/execute.py" %}

```python
from ice.recipe import recipe
from ice.recipes.primer.answer_by_dispatch.prompt import *


async def select_action(question: str) -> Action:
    prompt = make_action_selection_prompt(question)
    choices = tuple(str(i) for i in range(1, 6))
    choice_probs, _ = await recipe.agent().classify(prompt=prompt, choices=choices)
    best_choice = max(choice_probs.items(), key=lambda x: x[1])[0]
    return action_types[int(best_choice) - 1]


async def answer_by_dispatch(question: str = "How many people live in Germany?") -> str:
    action = await select_action(question)
    result = await action.recipe(question=question)
    return result


recipe.main(answer_by_dispatch)
```

{% endcode %}

Let's try it with our examples above:

{% code overflow="wrap" %}

```
$ python answer_by_dispatch.py --question "How many people live in Germany?"

The current population of Germany is 84,370,487 as of Monday, September 12, 2022, based on Worldometer elaboration of the latest United Nations data.
```

{% endcode %}

```
$ python answer_by_dispatch.py --question "What is sqrt(2^8)?"

16.0
```

{% code overflow="wrap" %}

```
$ python answer_by_dispatch.py --question "Is transhumanism desirable?"

It is up to each individual to decide whether or not they believe transhumanism is desirable.
```

{% endcode %}

These are better answers than we'd get without augmentation.

### Exercises

1. Suppose that actions are taking place within the context of a long document. Add an action type for searching for a particular phrase in the document and returning the results.a
2. Add an action type for debate:

{% code overflow="wrap" %}

```python
Action(
  name="Debate",
  description='Run a debate. This is helpful if the question is a pro/con question that involves involves different perspectives, arguments, and evidence, such as "Should marijuana be legalized?" or "Is veganism better for the environment?".'
)
```

{% endcode %}

<details>

<summary>Get feedback on exercise solutions</summary>

If you want feedback on your exercise solutions, submit them through [this form](https://docs.google.com/forms/d/e/1FAIpQLSdNNHeQAT7GIzn4tdsVYCkrVEPMNaZmBFkZCAJdvTvLzUAnzQ/viewform). We—the team at Ought—are happy to give our quick take on whether you missed any interesting ideas.

</details>
