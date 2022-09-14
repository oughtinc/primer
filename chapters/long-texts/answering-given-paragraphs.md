---
description: Answering the question given the top paragraphs with subrecipes
---

# Answering given paragraphs

Now all we have to do is combine the paragraph finder with the question-answering recipe we built earlier on.

We could just paste in the code from [the question-answering recipe](../question-answering.md#answering-questions-about-short-texts). However, we can also _directly_ reuse it as a subrecipe. If you have the code in `qa.py`, we can directly import and use it:

```python
from ice.recipe import recipe
from ice.paper import Paper, Paragraph
from ice.utils import map_async
from qa import answer


def make_classification_prompt(paragraph: Paragraph, question: str) -> str:
    return f"""Here is a paragraph from a research paper: "{paragraph}"

Question: Does this paragraph answer the question '{question}'? Say Yes or No.
Answer:"""


async def classify_paragraph(paragraph: Paragraph, question: str) -> float:
    choice, choice_prob, _ = await recipe.agent().classify(
        prompt=make_classification_prompt(paragraph, question),
        choices=(" Yes", " No"),
    )
    return choice_prob if choice == " Yes" else 1 - choice_prob

async def get_relevant_paragraphs(
    paper: Paper, question: str, top_n: int = 3
) -> list[Paragraph]:
    probs = await map_async(
        paper.paragraphs, lambda par: classify_paragraph(par, question)
    )
    sorted_pairs = sorted(
        zip(paper.paragraphs, probs), key=lambda x: x[1], reverse=True
    )
    return [par for par, prob in sorted_pairs[:top_n]]

@recipe.main
async def answer_for_paper(
    *, paper: Paper, question: str = "What was the study population?"
):
    relevant_paragraphs = await get_relevant_paragraphs(paper, question)
    relevant_str = "\n\n".join(str(p) for p in relevant_paragraphs)
    answer = await answer(context=relevant_str, question=question)
    return answer
```

Running the same command again...

```shell
python paperqa.py -t --paper papers/keenan-2018.pdf
```

... you should get an answer like this:

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
