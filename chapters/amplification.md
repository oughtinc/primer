---
description: Recursive question-answering
---

# Amplification

Amplification is the idea that agents can make subcalls to copies of themselves to inform how to answer a question or make a decision.

### Asking subquestions

Let's start by making a recipe that returns subquestions given a question:

```python
from ice.recipe import Recipe


def make_subquestion_prompt(question: str) -> str:
    return f"""Decompose the following question into 2-5 subquestions that would help you answer the question. Make the questions stand alone, so that they can be answered without the context of the original question.

Question: "{question}"
Subquestions:
-""".strip()


class Subquestions(Recipe):
    async def run(self, question: str = "What is the effect of creatine on cognition?"):
        prompt = make_subquestion_prompt(question)
        subquestions_text = await self.agent().answer(
            prompt=prompt, multiline=True, max_tokens=100
        )
        subquestions = [line.strip("- ") for line in subquestions_text.split("\n")]
        return subquestions
```

If we save this as `subquestions.py` and run it...

```shell
./scripts/run-recipe.sh -r subquestions.py -t
```

...we get:

```python
[
    'What is creatine?',
    'What is cognition?',
    'How does creatine affect cognition?',
    'What are the benefits of creatine on cognition?',
    'What are the side effects of creatine on cognition?'
]
```

### Answering subquestions

Now we want to use the subquestions recipe to help a question-answerer like the one we built early on in this tutorial. We can start with the question-answerer we built earlier and modify it as follows:

1. Add a call to the subquestions recipe to generate subquestions
2. Use `map_async` to answer all the subquestions in parallel
3. Provide answers from subquestions as advice in the overall question-answering prompt

Let's start with (1) and (2), reusing the subquestions subrecipe:

```python
from ice.recipe import Recipe
from ice.utils import map_async
from subquestions import Subquestions


def make_qa_prompt(question: str) -> str:
    return f"""Answer the following question:

Question: "{question}"
Answer: "
""".strip()


class AmplifiedQA(Recipe):

    async def answer(self, question: str) -> str:
        prompt = make_qa_prompt(question)
        answer = (await self.agent().answer(prompt=prompt, max_tokens=100)).strip('" ')
        return answer

    async def run(self, question: str = "What is the effect of creatine on cognition?"):
         subquestions = await Subquestions().run(question=question)
         subanswers = await map_async(subquestions, self.answer)
         return list(zip(subquestions, subanswers))
```

If we run this, we get back a list of subquestions and their answers:

{% code overflow="wrap" %}
```python
[
    (
        'What is creatine?',
        'Creatine is a nitrogenous organic acid that helps supply energy to cells, primarily in the muscles.'
    ),
    (
        'What is cognition?',
        'Cognition is the mental action or process of acquiring knowledge and understanding through thought, experience, and the senses.'
    ),
    (
        'How does creatine affect cognition?',
        'Creatine is a dietary supplement that is often used by athletes to improve their performance. Some research has suggested that it may also improve cognitive function, but the evidence is mixed. Some studies have found that creatine can improve memory and reaction time, while others have found no significant effects.'
    ),
    (
        'What are the benefits of creatine on cognition?',
        'Creatine has been shown to improve cognitive function in people with certain medical conditions, such as Parkinson’s disease and Alzheimer’s disease. It has also been shown to improve cognitive function in healthy adults.'
    ),
    (
        'What are the side effects of creatine on cognition?',
        'There is no definitive answer to this question as the research on the topic is inconclusive. Some studies suggest that creatine may improve cognitive function, while other studies have found no significant effects. More research is needed to determine the potential cognitive effects of creatine.'
    )
]
```
{% endcode %}

### One-step amplification: Answering given subquestion answers

We need an equivalent of `make_qa_prompt` that optionally takes a list of subquestions and answers and provides those in the prompt. Let's introduce a type `Subs` for pairs of questions and answers and extend `make_qa_prompt` to use it if given:

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

```python
class AmplifiedQA(Recipe):
    async def run(self, question: str = "What is the effect of creatine on cognition?"):
        subs = await self.get_subs(question)
        answer = await self.answer(question=question, subs=subs)
        return answer

    async def get_subs(self, question: str) -> Subs:
        subquestions = await Subquestions().run(question=question)
        subanswers = await map_async(subquestions, self.answer)
        return list(zip(subquestions, subanswers))

    async def answer(self, question: str, subs: Subs = []) -> str:
        prompt = make_qa_prompt(question, subs=subs)
        answer = (await self.agent().answer(prompt=prompt, max_tokens=100)).strip('" ')
        return answer
```

If we run it with

```shell
scripts/run-recipe.sh -r amplified_qa.py -t
```

we get:

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

### Recursive amplification

Now we'd like to generalize the recipe above so that we can run it at different depths:

* Depth 0: Just answer the question, no subquestions
* Depth 1: One layer of subquestions
* Depth 2: Use subquestions when answering subquestions
* Etc.

To do this, we adda `depth` parameter to `run` and `get_subs` and only get subquestions if we're at depth > 0. This simplifies the amplification recipe to:

```python
class AmplifiedQA(Recipe):
    async def run(self, question: str = "What is the effect of creatine on cognition?", depth: int = 1):
        subs = await self.get_subs(question, depth - 1) if depth > 0 else []
        prompt = make_qa_prompt(question, subs=subs)
        answer = (await self.agent().answer(prompt=prompt, max_tokens=100)).strip('" ')
        return answer

    async def get_subs(self, question: str, depth: int) -> Subs:
        subquestions = await Subquestions().run(question=question)
        subanswers = await map_async(subquestions, lambda q: self.run(q, depth))
        return list(zip(subquestions, subanswers))
```

Now we have a parameterized recipe that we can run at different depths:

#### Depth 0

```shell
scripts/run-recipe.sh -r amplification.py -t --args '{"depth": 0}'
```

{% code overflow="wrap" %}
```
Creatine has been shown to improve cognition in people with Alzheimer's disease and other forms of dementia.
```
{% endcode %}

#### Depth 1

```shell
scripts/run-recipe.sh -r amplification.py -t --args '{"depth": 1}'
```

{% code overflow="wrap" %}
```
The effect of creatine on cognition is mixed. Some studies have found that creatine can help improve memory and reaction time, while other studies have found no significant effects. It is possible that the effects of creatine on cognition may vary depending on the individual.
```
{% endcode %}

#### Depth 2

```shell
scripts/run-recipe.sh -r amplification.py -t --args '{"depth": 2}'
```

{% code overflow="wrap" %}
```
The effect of creatine on cognition is inconclusive. Some studies have found that creatine can improve cognitive function in healthy adults, while other studies have found no significant effects. More research is needed to determine the potential cognitive benefits of creatine.
```
{% endcode %}

### Exercises

1. Right now we're answering subquestions without the context of the question they're intended to help with. Provide the question (or questions) that are further up in the hierarchy as additional context to the model.
2. Running subquestions in parallel is nice because it's fast, but has the disadvantage that the answers to later subquestions can't inform what question to ask next. Modify the recipe to support sequential choice of questions based on earlier responses.
3. Now make the recipe from step 1 adaptive: Let the model decide whether to answer or whether to ask another subquestion. If you don't limit the depth, what is the effective depth that the model ends up for different questions?
