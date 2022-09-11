---
description: The simplest recipe
---

# Hello World

Let's first get used to the infrastructure for writing, running, and debugging recipes:

Create a file `hello.py`:

```python
from ice.recipe import Recipe

class HelloWorld(Recipe):
    async def run(self):
        return "Hello world!"
```

Run the recipe:

```shell
scripts/run-recipe.sh -r hello.py -t
```

This will run the recipe, creating an execution trace (`-t`).

On the terminal, after a few lines about Docker and the trace link, you should see this:

```
Hello world!
```

If you follow the link, in your browser you should see a function node that you can click on, expand, and inspect inputs/outputs and source code.

<details>

<summary>The recipe, line by line</summary>

* We're inhering from the `Recipe` class because that will give us automatic tracing of all async methods for debugging. Synchronous methods are assumed to be simple and fast, and not worth tracing.
* `run` is the name of the method that is called when a recipe is run by `run-recipe`.
* Most recipe methods, including `run`, will be async so that language model calls are parallelized as much as possible.
* Different recipes take different arguments, which will be provided as keyword arguments to `run`. This recipe doesn't use any arguments.

</details>

### Exercises

1. Add another method to `HelloWorld` and call it from `run`. Does it show up in the trace? What if you make it async and call it as `result = await self.my_function()`?
