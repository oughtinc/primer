# Checking reasoning steps

Let's change the interface of the verifier so that it doesn't just take an answer, but also a sequence of reasoning steps leading up to it. This way, we can check each step independently and get a probability that it's correct.

## **Representing and rendering reasoning steps**

First, let's represent reasoning steps as a list (so that we can more easily manipulate them programmatically) and make a function to render them as a string (so that we can use them in prompts):

{% code overflow="wrap" %}
```python
from ice.recipe import Recipe
from ice.utils import map_async

DEFAULT_QUESTION = "Beth bakes 4x 2 dozen batches of cookies in a week. If these cookies are shared amongst 16 people equally, how many cookies does each person consume?"

DEFAULT_STEPS = [
    "Beth bakes 4x 2 dozen batches of cookies for a total of 4*2 = 8 dozen cookies",
    "There are 12 cookies in a dozen and she makes 8 dozen cookies for a total of 12*8 = 96 cookies",
    "She splits the 96 cookies equally amongst 16 people so they each eat 96/16 = 6 cookies",
    "So, the final answer is 6 cookies per person.",
]


def render_steps(steps: list[str]) -> str:
    return "\n".join(f"{i}. {step}" for (i, step) in enumerate(steps, start=1))
```
{% endcode %}

If we run `render_steps(DEFAULT_STEPS)`, we get back the original numbered list:

{% code overflow="wrap" %}
```
1. Beth bakes 4x 2 dozen batches of cookies for a total of 4*2 = 8 dozen cookies
2. There are 12 cookies in a dozen and she makes 8 dozen cookies for a total of 12*8 = 96 cookies
3. She splits the 96 cookies equally amongst 16 people so they each eat 96/16 = 6 cookies
4. So, the final answer is 6 cookies per person.
```
{% endcode %}

## **Verifying a step**

Given a list of steps, let's first think about how we can verify the last step, assuming all previous ones are correct.

This is effectively the same as the global verifier above, except that we need to render the steps befoer we make the prompt. We'll also already factor out the step verification into a function `check_step` so that we can reuse it later.

{% code overflow="wrap" %}
```python
def make_verification_prompt(question: str, steps: list[str]) -> str:
    return f"""Consider this question: "{question}"

Here are the first few steps of an answer:

{render_steps(steps)}

Q: Is step {len(steps)} correct, assuming that the previous steps are correct? Say "A: Yes" or "A: No".
A:"""


class Verifier(Recipe):
    async def run(
        self, question: str = DEFAULT_QUESTION, steps: list[str] = DEFAULT_STEPS
    ):
        return await self.check_step(question=question, steps=steps)

    async def check_step(self, question: str, steps: list[str]) -> float:
        """
        Return the probability that the step is correct
        """
        prompt = make_verification_prompt(question=question, steps=steps)
        answer, answer_p, _ = await self.agent().classify(
            prompt=prompt, choices=[" Yes", " No"]
        )
        p_correct = answer_p if answer == " Yes" else 1 - answer_p
        return p_correct
```
{% endcode %}

If we run this with the default question and steps:

```shell
scripts/run-recipe.sh -r verify_last_step.py -t
```

We get:

```
0.8373182599538002
```

Note that (as we'd expect) this probability of the last step being correct is significantly higher than the probability the model assigned to the entire answer being correct.

## **Verifying all steps**

To verify all steps, we simply replace `run` with an (async) map over all prefixes of steps:

```python
class Verifier(Recipe):
    async def run(
        self, question: str = DEFAULT_QUESTION, steps: list[str] = DEFAULT_STEPS
    ):
        """
        For each prefix of 1..n steps, check if the nth step is correct.
        """
        step_indices = list(range(1, len(steps) + 1))
        step_probs = await map_async(
            step_indices,
            lambda index: self.check_step(question=question, steps=steps[:index]),
        )
        return list(zip(step_probs, steps))

    async def check_step(self, question: str, steps: list[str]) -> float:
        """
        Return the probability that the step is correct
        """
        prompt = make_verification_prompt(question=question, steps=steps)
        answer, answer_p, _ = await self.agent().classify(
            prompt=prompt, choices=[" Yes", " No"]
        )
        p_correct = answer_p if answer == " Yes" else 1 - answer_p
        return p_correct
```

Instead of just returning the probabilities, we return pairs of probabilities and steps to make the result easier to read. It looks like this:

{% code overflow="wrap" %}
```python
[
    (
        0.7626456256640988,
        'Beth bakes 4x 2 dozen batches of cookies for a total of 4*2 = 8 dozen cookies'
    ),
    (
        0.5670302526651101,
        'There are 12 cookies in a dozen and she makes 8 dozen cookies for a total of 12*8 = 96 cookies'
    ),
    (
        0.5074000096282046,
        'She splits the 96 cookies equally amongst 16 people so they each eat 96/16 = 6 cookies'
    ),
    (
        0.827695609429836, 
        'So, the final answer is 6 cookies per person.'
    )
]
```
{% endcode %}

The more difficult the math, the lower the probability the model assigns to the step being correct.

## Exercises

1. How could you use the probabilities we get for each step? One idea is to use a model to resample steps that are wrong. Can you use this to answer questions more correctly?
2. If we multiply the probabilities above to get the probability that the argument overall is correct, we get $$0.76 \cdot 0.57 \cdot 0.51 \cdot 0.83 = 0.18$$. In general, the more steps, the lower we should expect the product probability to be. If we can't get high probability by [just checking the answer](checking-answers.md), and we can't get it by checking many steps, how can we ever confidently conclude that an answer is correct? What does your answer to this question mean for how to implement and check reasoning using language models?