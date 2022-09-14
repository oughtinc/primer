---
description: The simplest recipe
---

# Hello World

Let's first get used to the infrastructure for writing, running, and debugging recipes:

Create a file `hello.py`:

```python
from ice.recipe import recipe

@recipe.main
async def say_hello(self):
    return "Hello world!"
```

Run the recipe:

```shell
python hello.py -t
```

This will run the recipe, creating an execution trace (`-t`).

On the terminal, after a few lines about Docker and the trace link, you should see this:

```
Hello world!
```

If you follow the link, in your browser you should see a function node that you can click on, expand, and inspect inputs/outputs and source code.

<details>

<summary>The recipe, line by line</summary>

* We use `@recipe.main` to denote the recipe entry point and to automatically trace all global async functions that were defined in this file. Synchronous functions are assumed to be simple and fast, and not worth tracing.
* The entry point must be the last function in the file, and must be `async`.
* Most recipe functions will be async so that language model calls are parallelized as much as possible.
* Different recipes take different arguments, which will be provided as keyword arguments to the entry point. This recipe doesn't use any arguments.

</details>

### Exercises

1. Add another function and call it from `say_hello`. Does it show up in the trace? What if you make it async and call it as `result = await my_function()`?
