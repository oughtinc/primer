---
description: Paper Q&A
---

# Long Texts

Ultimately we'd like for language models to help us make sense of more information than we can read and understand ourselves. As a small step in that direction, let's make a recipe that answers questions about a paper.

### Loading papers in recipes

ICE has built-in functionality for parsing and loading papers, and includes some example papers in its `papers` folder. Here's a minimal recipe that loads a paper and prints out the first paragraph (often the abstract):

```python
from ice.recipe import Recipe
from ice.paper import Paper

class PaperQA(Recipe):
    async def run(self, paper: Paper):
        return paper.paragraphs[0]
```

If you have this recipe as `paperqa.py`, you can run it as follows, providing a paper as input (`-i`):

```shell
scripts/run-recipe.sh -r paperqa.py -t -i papers/keenan-2018.pdf
```

You'll see a result like this:

{% code overflow="wrap" %}
```python
Paragraph(sentences=['We hypothesized that mass distribution of a broad-spectrum antibiotic agent to preschool children would reduce mortality in areas of sub-Saharan Africa that are currently far from meeting the Sustainable Development Goals of the United Nations.'], sections=[Section(title='Abstract', number=None)], section_type='abstract')
```
{% endcode %}

Note that:

* Papers are represented as lists of paragraphs
* Paragraphs are represented as lists of sentences
* Each paragraph has information about which section it's from

Feel free to try it with your own papers!

### Finding the most relevant paragraphs

In this recipe we'll take a simple approach:

1. Classify for each paragraph whether it answers the question or not
2. Take the paragraphs with the highest probability of answering the question and ask a model to answer the question given those paragraphs. For this step we can use the QA recipe from above.

#### Classifying individual paragraphs using `classify`

Let's start by just classifying whether the first paragraph answers a question. To do this, we'll use a new agent method, `classify`. It takes a prompt and a list of choices, and returns a choice, a choice probability, and for some agent implementations an explanation.

Our single-paragraph classifier looks like this:

```python
from ice.recipe import Recipe
from ice.paper import Paper, Paragraph

def make_prompt(paragraph: Paragraph, question: str) -> str:
    return f"""
Here is a paragraph from a research paper: "{paragraph}"

Question: Does this paragraph answer the question '{question}'? Say Yes or No.
Answer:""".strip()

class PaperQA(Recipe):

    async def classify_paragraph(self, paragraph: Paragraph, question: str) -> float:
        choice, choice_prob, _ = await self.agent().classify(
            prompt=make_prompt(paragraph, question),
            choices=(" Yes", " No"),
        )
        return choice_prob if choice == " Yes" else 1 - choice_prob

    async def run(self, paper: Paper):
        paragraph = paper.paragraphs[0]
        question = kw.get("question", "What was the study population?")
        return await self.classify_paragraph(paragraph, question)
```

Save it to `paperqa.py` and run it on a paper:

```shell
./scripts/run-recipe.sh -r paperqa.py -t -i papers/keenan-2018.pdf
```

You should see a result like this:

```python
0.024985359096987403
```

According to the model, the first paragraph is unlikely to answer the question.

#### Classifying all paragraphs in parallel with `map_async`

To find the most relevant paragraphs, we map the paragraph classifier over all paragraphs and get the most likely ones.

For mapping, we use the utility `map_async` which runs the language model calls in parallel:

```python
from ice.recipe import Recipe
from ice.paper import Paper, Paragraph
from ice.utils import map_async

def make_prompt(paragraph: Paragraph, question: str) -> str:
    return f"""Here is a paragraph from a research paper: "{paragraph}"

Question: Does this paragraph answer the question '{question}'? Say Yes or No.
Answer:"""

class PaperQA(Recipe):

    async def classify_paragraph(self, paragraph: Paragraph, question: str) -> float:
        choice, choice_prob, _ = await self.agent().classify(
            prompt=make_prompt(paragraph, question),
            choices=(" Yes", " No"),
        )
        return choice_prob if choice == " Yes" else 1 - choice_prob

    async def run(self, paper: Paper):
        paragraph = paper.paragraphs[0]
        question = kw.get("question", "What was the study population?")
        probs = await map_async(paper.paragraphs, lambda par: self.classify_paragraph(par, question))
        return probs
```

If you run the same command as above, you will now see a list of probabilities, one for each paragraph:

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

```python
from ice.recipe import Recipe
from ice.paper import Paper, Paragraph
from ice.utils import map_async


def make_classification_prompt(paragraph: Paragraph, question: str) -> str:
    return f"""Here is a paragraph from a research paper: "{paragraph}"

Question: Does this paragraph answer the question '{question}'? Say Yes or No.
Answer:"""


class PaperQA(Recipe):
    async def classify_paragraph(self, paragraph: Paragraph, question: str) -> float:
        choice, choice_prob, _ = await self.agent().classify(
            prompt=make_classification_prompt(paragraph, question),
            choices=(" Yes", " No"),
        )
        return choice_prob if choice == " Yes" else 1 - choice_prob

    async def run(
        self, paper: Paper, question: str, top_n: int = 3
    ) -> list[Paragraph]:
        probs = await map_async(
            paper.paragraphs, lambda par: self.classify_paragraph(par, question)
        )
        sorted_pairs = sorted(
            zip(paper.paragraphs, probs), key=lambda x: x[1], reverse=True
        )
        return [par for par, prob in sorted_pairs[:top_n]]
```

Running the same command again...

```shell
./scripts/run-recipe.sh -r paperqa.py -t -i papers/keenan-2018.pdf
```

...we indeed get paragraphs that answer the question who the study population was!

{% code overflow="wrap" %}
```python
[
    Paragraph(sentences=['A total of 1624 communities were eligible for inclusion in the trial on the basis of the most recent census (Fig. 1 ).', 'A random selection of 1533 communities were included in the current trial, and the remaining 91 were enrolled in smaller parallel trials at each site, in which additional microbiologic, anthropometric, and adverse-event data were collected.', 'In Niger, 1 community declined to participate and 20 were excluded because of census inaccuracies.', 'No randomization units were lost to follow-up after the initial census.'], sections=[Section(title='Participating Communities', number=None)], section_type='main'),
    ...
]
```
{% endcode %}

### Answering the question given the top paragraphs with subrecipes

Now all we have to do is combine the paragraph finder with the question-answering recipe we built earlier on.

We could just paste in the code from the other recipe. However, we can also _directly_ reuse it as a subrecipe. If you have the code in `qa.py`, we can directly import and use it:

```python
from ice.recipe import Recipe
from ice.paper import Paper, Paragraph
from ice.utils import map_async
from qa import QA


def make_classification_prompt(paragraph: Paragraph, question: str) -> str:
    return f"""Here is a paragraph from a research paper: "{paragraph}"

Question: Does this paragraph answer the question '{question}'? Say Yes or No.
Answer:"""


class PaperQA(Recipe):
    async def classify_paragraph(self, paragraph: Paragraph, question: str) -> float:
        choice, choice_prob, _ = await self.agent().classify(
            prompt=make_classification_prompt(paragraph, question),
            choices=(" Yes", " No"),
        )
        return choice_prob if choice == " Yes" else 1 - choice_prob

    async def get_relevant_paragraphs(
        self, paper: Paper, question: str, top_n: int = 3
    ) -> list[Paragraph]:
        probs = await map_async(
            paper.paragraphs, lambda par: self.classify_paragraph(par, question)
        )
        sorted_pairs = sorted(
            zip(paper.paragraphs, probs), key=lambda x: x[1], reverse=True
        )
        return [par for par, prob in sorted_pairs[:top_n]]

    async def run(
        self, paper: Paper, question: str = "What was the study population?"
    ):
        relevant_paragraphs = await self.get_relevant_paragraphs(paper, question)
        relevant_str = "\n\n".join(str(p) for p in relevant_paragraphs)
        answer = await QA().run(context=relevant_str, question=question)
        return answer
```

If you run it with the same command as above, you should get an answer like this:

{% code overflow="wrap" %}
```
The study population was children 1 to 59 months of age who weighed at least 38
```
{% endcode %}

Take a look at the trace to see how it all fits together.

### Exercises

1. We're taking a fixed number of paragraphs (3) and sticking them into the prompt. This will sometimes result in prompt space not being used well, and sometimes it will overflow. Modify the recipe to use as many paragraphs as can fit into the prompt. (Hint: A prompt for current models has space for 2048 tokens. A token is about 3.5 characters.)
2. We're classifying paragraphs individually, but it could be better to do ranking by showing the model pairs of paragraphs and ask it which better answers the question. Implement this as an alternative.
3. (Advanced) Implement debate with agents that have access to the same paper, and let agents provide quotes from the paper that can't be faked. Does being able to refer to ground truth quotes favor truth in debate?
