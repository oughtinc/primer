---
description: How to install and run the Interaction Composition Explorer
---

# Before We Start

The recipes in this primer are implemented using the [Interactive Composition Explorer](https://github.com/oughtinc/ice) (ICE). If you’d like to follow along with the implementation (strongly recommended), set it up first.

## Requirements

ICE requires Python 3.9, 3.10, or 3.11. If you don't have a supported version of Python installed, we recommend using [pyenv](https://github.com/pyenv/pyenv) to install a supported Python version and manage multiple Python versions.

If you use Windows, you'll need to run ICE inside of [WSL](https://learn.microsoft.com/en-us/windows/wsl/install).

## Run ICE

As part of general good Python practice, consider first creating and activating a [virtual environment](https://docs.python.org/3/library/venv.html) to avoid installing ICE 'globally'. For example:

```shell
python3.10 -m venv venv
source venv/bin/activate
```

Install ICE:

```shell
pip install ought-ice
```

The first time you run a recipe that uses `recipe.agent()`, you will be prompted for an [OpenAI API key](https://beta.openai.com/account/api-keys) that will automatically be stored in `~/.ought-ice/.env`.
