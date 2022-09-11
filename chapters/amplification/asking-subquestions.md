# Asking subquestions



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