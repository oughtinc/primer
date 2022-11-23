---
description: How to install and run the Interaction Composition Explorer
---

# Before We Start

The recipes in this primer are implemented using the [Interactive Composition Explorer](https://github.com/oughtinc/ice) (ICE). If you’d like to follow along with the implementation (strongly recommended), set it up first.

## Requirements

ICE requires Python 3.10.

If you use Windows, you'll need to run ICE inside of [WSL](https://learn.microsoft.com/en-us/windows/wsl/install).

## Run ICE

Install ICE:

```shell
pip install ought-ice
```

Obtain an [`OPENAI_API_KEY`](https://beta.openai.com/account/api-keys) and create an `.env` file containing it in the ICE folder:

{% code title="~/.ought-ice/.env" %}

```shell
OPENAI_API_KEY=sk-...f8 # Replace with your API key.
```

{% endcode %}

Start ICE in its own terminal and leave it running:

```shell
python -m ice.server
```
