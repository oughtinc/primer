# Recursive amplification

Now we'd like to generalize the recipe above so that we can run it at different depths:

- Depth 0: Just answer the question, no subquestions
- Depth 1: One layer of subquestions
- Depth 2: Use subquestions when answering subquestions
- Etc.

To do this, we adda `depth` parameter to `run` and `get_subs` and only get subquestions if we're at depth > 0. This simplifies the amplification recipe to:

```python
@recipe.main
async def answer_by_amplification(question: str = "What is the effect of creatine on cognition?", depth: int = 1):
    subs = await get_subs(question, depth - 1) if depth > 0 else []
    prompt = make_qa_prompt(question, subs=subs)
    answer = (await recipe.agent().answer(prompt=prompt, max_tokens=100)).strip('" ')
    return answer

async def get_subs(question: str, depth: int) -> Subs:
    subquestions = await ask_subquestions(question=question)
    subanswers = await map_async(subquestions, lambda q: run(q, depth))
    return list(zip(subquestions, subanswers))
```

Now we have a parameterized recipe that we can run at different depths:

#### Depth 0

```shell
python amplification.py -t --depth 0
```

{% code overflow="wrap" %}

```
Creatine has been shown to improve cognition in people with Alzheimer's disease and other forms of dementia.
```

{% endcode %}

#### Depth 1

```shell
python amplification.py -t --depth 1
```

{% code overflow="wrap" %}

```
The effect of creatine on cognition is mixed. Some studies have found that creatine can help improve memory and reaction time, while other studies have found no significant effects. It is possible that the effects of creatine on cognition may vary depending on the individual.
```

{% endcode %}

#### Depth 2

```shell
python amplification.py -t --depth 2
```

{% code overflow="wrap" %}

```
The effect of creatine on cognition is inconclusive. Some studies have found that creatine can improve cognitive function in healthy adults, while other studies have found no significant effects. More research is needed to determine the potential cognitive benefits of creatine.
```

{% endcode %}

## Exercises

1. Right now we're answering subquestions without the context of the question they're intended to help with. Provide the question (or questions) that are further up in the hierarchy as additional context to the model.
2. Running subquestions in parallel is nice because it's fast, but has the disadvantage that the answers to later subquestions can't inform what question to ask next. Modify the recipe to support sequential choice of questions based on earlier responses.
3. Now make the recipe from step 1 adaptive: Let the model decide whether to answer or whether to ask another subquestion. If you don't limit the depth, what is the effective depth that the model ends up for different questions?
