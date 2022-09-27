---
description: Paragraph-wise classification with parallelism
---

# Finding relevant paragraphs

## Classifying individual paragraphs using `classify`

Let's start by just classifying whether the first paragraph answers a question. To do this, we'll use a new agent method, `classify`. It takes a prompt and a list of choices, and returns a choice, a choice probability, and for some agent implementations an explanation.

Our single-paragraph classifier looks like this:

{% code title="paper_qa_class.py" %}
```python
from ice.paper import Paper
from ice.paper import Paragraph
from ice.recipe import recipe


def make_prompt(paragraph: Paragraph, question: str) -> str:
    return f"""
Here is a paragraph from a research paper: "{paragraph}"

Question: Does this paragraph answer the question '{question}'? Say Yes or No.
Answer:""".strip()


async def classify_paragraph(paragraph: Paragraph, question: str) -> float:
    choice_probs, _ = await recipe.agent().classify(
        prompt=make_prompt(paragraph, question),
        choices=(" Yes", " No"),
    )
    return choice_probs.get(" Yes", 0.0)


async def answer_for_paper(paper: Paper, question: str):
    paragraph = paper.paragraphs[0]
    return await classify_paragraph(paragraph, question)


recipe.main(answer_for_paper)
```
{% endcode %}

Save it and run it on a paper:

{% code overflow="wrap" %}
```shell
python paper_qa_class.py --paper papers/keenan-2018.pdf --question "What was the study population?"
```
{% endcode %}

You should see a result like this:

```python
0.024985359096987403
```

According to the model, the first paragraph is unlikely to answer the question.

## Classifying all paragraphs in parallel with `map_async`

To find the most relevant paragraphs, we map the paragraph classifier over all paragraphs and get the most likely ones.

For mapping, we use the utility `map_async` which runs the language model calls in parallel:

{% code title="paper_qa_classes.py" %}
```python
from ice.paper import Paper
from ice.paper import Paragraph
from ice.recipe import recipe
from ice.utils import map_async


def make_prompt(paragraph: Paragraph, question: str) -> str:
    return f"""Here is a paragraph from a research paper: "{paragraph}"

Question: Does this paragraph answer the question '{question}'? Say Yes or No.
Answer:"""


async def classify_paragraph(paragraph: Paragraph, question: str) -> float:
    choice_probs, _ = await recipe.agent().classify(
        prompt=make_prompt(paragraph, question),
        choices=(" Yes", " No"),
    )
    return choice_probs.get(" Yes", 0.0)


async def answer_for_paper(
    paper: Paper, question: str = "What was the study population?"
):
    probs = await map_async(
        paper.paragraphs, lambda par: classify_paragraph(par, question)
    )
    return probs


recipe.main(answer_for_paper)
```
{% endcode %}

You will now see a list of probabilities, one for each paragraph:

```python
[
    0.024381454349145293,
    0.24823367447526778,
    0.21119208211186247,
    0.07488850139282821,
    0.16529937276656714,
    0.46596912974665494,
    0.09871877171479271,
    ...
    0.06523843237521842,
    0.041946178310281246,
    0.03264635093381785,
    0.023112249840077093,
    0.0018325902029144858,
    0.15962813987814772
]
```

Now all we need to do is add a utility function for looking up the paragraphs with the highest probabilities:

{% code title="paper_qa_ranker.py" %}
```python
from ice.paper import Paper
from ice.paper import Paragraph
from ice.recipe import recipe
from ice.utils import map_async


def make_classification_prompt(paragraph: Paragraph, question: str) -> str:
    return f"""Here is a paragraph from a research paper: "{paragraph}"

Question: Does this paragraph answer the question '{question}'? Say Yes or No.
Answer:"""


async def classify_paragraph(paragraph: Paragraph, question: str) -> float:
    choice_probs, _ = await recipe.agent().classify(
        prompt=make_classification_prompt(paragraph, question),
        choices=(" Yes", " No"),
    )
    return choice_probs.get(" Yes", 0)


async def answer_for_paper(
    paper: Paper, question: str, top_n: int = 3
) -> list[Paragraph]:
    probs = await map_async(
        paper.paragraphs, lambda par: classify_paragraph(par, question)
    )
    sorted_pairs = sorted(
        zip(paper.paragraphs, probs), key=lambda x: x[1], reverse=True
    )
    return [par for par, prob in sorted_pairs[:top_n]]


recipe.main(answer_for_paper)
```
{% endcode %}

Running the same command again...

{% code overflow="wrap" %}
```shell
python paper_qa_ranker.py --paper papers/keenan-2018.pdf --question "What was the study population?"
```
{% endcode %}

...we indeed get paragraphs that answer the question who the study population was!

{% code overflow="wrap" %}
```python
[
    Paragraph(sentences=['A total of 1624 communities were eligible for inclusion in the trial on the basis of the most recent census (Fig. 1 ).', 'A random selection of 1533 communities were included in the current trial, and the remaining 91 were enrolled in smaller parallel trials at each site, in which additional microbiologic, anthropometric, and adverse-event data were collected.', 'In Niger, 1 community declined to participate and 20 were excluded because of census inaccuracies.', 'No randomization units were lost to follow-up after the initial census.'], sections=[Section(title='Participating Communities', number=None)], section_type='main'),
    ...
]
```
{% endcode %}
