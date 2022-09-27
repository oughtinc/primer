---
description: Executing code for more accurate computation
---

# Interpreters

Sometimes the limitation isn't factual knowledge, but ability to do computation.

For example, if we ask [the basic question-answerer](../question-answering/q-and-a-without-context.md) "What is 578921 days \* 12312 miles/day?":

```shell
python qa_simple.py --question "What is 578921 days * 12312 miles/day?"
```

we get:

```python
7223849252 miles
```

This is similar to the correct answer `7127675352 miles`, but not the same.

## Evaluating Python expressions

Let's add a method for evaluating Python expressions:

{% code title="eval_direct.py" %}
```python
from ice.recipe import recipe


def eval_python(expression: str) -> str:
    try:
        result = eval(expression)
    except Exception as e:
        result = f"Error: {e}"
    return str(result)


async def answer_by_computation(question: str):
    return eval_python(question)


recipe.main(answer_by_computation)
```
{% endcode %}

This works as expected for expressions that are literally Python code:

```shell
python eval_direct.py --question "1 + 1"
```

```
2
```

Of course, it doesn't work for natural language questions that benefit from compute:

{% code overflow="wrap" %}

```shell
python eval_direct.py --question "What is 578921 days * 12312 miles/day?"
```

{% endcode %}

```
Error: invalid syntax (<string>, line 1)
```

So, we need to choose what to evaluate.

## Choosing what to evaluate

We make a prompt that asks the model what expression to enter into a Python interpreter to answer the question. We'll also print out the result of evaluating this expression:

{% code title="eval_selective.py" %}
```python
from ice.recipe import recipe


def make_computation_choice_prompt(question: str) -> str:
    return f"""You've been asked to answer the question "{question}".

You have access to a Python interpreter.

Enter an expression that will help you answer the question.
>>>"""


def eval_python(expression: str) -> str:
    try:
        result = eval(expression)
    except Exception as e:
        result = f"Error: {e}"
    return str(result)


async def choose_computation(question: str) -> str:
    prompt = make_computation_choice_prompt(question)
    answer = (await recipe.agent().answer(prompt=prompt)).strip('" ')
    return answer


async def eval_selective(question: str):
    expression = await choose_computation(question)
    result = eval_python(expression)
    return (expression, result)


recipe.main(eval_selective)
```
{% endcode %}

If we run this on our example, we get:

```
('578921 * 12312', '7127675352')
```

This is a helpful expression and result!

## Using the results of evaluation

Now all we need to do this provide this expression and result as additional context for the basic question-answerer.

{% code title="answer_by_computation.py" %}
```python
from ice.recipe import recipe


def make_computation_choice_prompt(question: str) -> str:
    return f"""You've been asked to answer the question "{question}".

You have access to a Python interpreter.

Enter an expression that will help you answer the question.
>>>"""


def make_compute_qa_prompt(question: str, expression: str, result: str) -> str:
    return f"""A recording of a Python interpreter session:

>>> {expression}: {result}

Answer the following question, using the Python session if helpful:

Question: "{question}"
Answer: "
""".strip()


def eval_python(expression: str) -> str:
    try:
        result = eval(expression)
    except Exception as e:
        result = f"Error: {e}"
    return str(result)


async def choose_computation(question: str) -> str:
    prompt = make_computation_choice_prompt(question)
    answer = (await recipe.agent().answer(prompt=prompt)).strip('" ')
    return answer


async def answer_by_computation(question: str):
    expression = await choose_computation(question)
    result = eval_python(expression)
    prompt = make_compute_qa_prompt(question, expression, result)
    answer = (await recipe.agent().answer(prompt=prompt, multiline=False)).strip('" ')
    return answer


recipe.main(answer_by_computation)
```
{% endcode %}

Rerunning our test case

{% code overflow="wrap" %}

```shell
python answer_by_computation.py --question "What is 578921 days * 12312 miles/day?"
```

{% endcode %}

we get the correct answer:

```
7127675352 miles
```

Another example:

> If I have $500 and get 3.7% interest over 16 years, what do I have at the end?

Running this:

{% code overflow="wrap" %}

```shell
python answer_by_computation.py --question "If I have $500 and get 3.7% interest over 16 years, what do I have at the end?"
```

{% endcode %}

We get:

{% code overflow="wrap" %}

```
If you have $500 and get 3.7% interest over 16 years, you will have $894.19 at the end.
```

{% endcode %}

In contrast, the basic question-answerer says "You would have $1,034,957.29 at the end."

## Exercises

1. Many questions can only be answered using longer algorithms in Python. Extend the code above to support multi-line Python programs ([example](https://twitter.com/sergeykarayev/status/1569377881440276481/photo/1)).
2. Another approach to (1) is to let the model "enter" multiple expressions into the interpreter. Extend the recipe to support this.

<details>

<summary>Get feedback on exercise solutions</summary>

If you want feedback on your exercise solutions, submit them through [this form](https://docs.google.com/forms/d/e/1FAIpQLSdNNHeQAT7GIzn4tdsVYCkrVEPMNaZmBFkZCAJdvTvLzUAnzQ/viewform). We—the team at Ought—are happy to give our quick take on whether you missed any interesting ideas.

</details>
