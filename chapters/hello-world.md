---
description: The simplest recipe
---

# Hello World

Let's first get used to the infrastructure for writing, running, and debugging recipes:

Create a file `hello.py`:

```python
from ice.recipe import recipe

async def say_hello():
    return "Hello world!"

recipe.main(say_hello)
```

Run the recipe [in Docker](../before-we-start.md#shell-into-the-docker-container):

```shell
python hello.py
```

This will run the recipe and save an execution trace.

On the terminal, you should see the trace link, then this:

```
Hello world!
```

If you follow the trace link, in your browser you should see a function node that you can click on, expand, and inspect inputs/outputs and source code.

<details>

<summary>The recipe, line by line</summary>

* We use `recipe.main` to denote the recipe entry point and to automatically trace all global async functions that were defined in this file. Synchronous functions are assumed to be simple and fast, and not worth tracing.
* `recipe.main` must appear at the bottom of the file.
* The entry point must be async.
* Most recipe functions will be async so that language model calls are parallelized as much as possible.
* Different recipes take different arguments, which will be provided as keyword arguments to the entry point. This recipe doesn't use any arguments.

</details>

### Exercises

1. Add another function and call it from `say_hello`. Does it show up in the trace? What if you make it async and call it as `result = await my_function()`?
