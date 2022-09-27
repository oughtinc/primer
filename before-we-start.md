---
description: How to install and run the Interaction Composition Explorer
---

# Before We Start

The recipes in this primer are implemented using the [Interactive Composition Explorer](https://github.com/oughtinc/ice) (ICE). If you'd like to follow along with the implementation (strongly recommended), set it up first.

## Prerequisites

Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)

## Setup

Clone ICE:

```shell
git clone https://github.com/oughtinc/ice.git
```

Add a `.env` file containing an `OPENAI_API_KEY` to the ICE folder. See [here](https://openai.com/api/) for information about the OpenAI API.

```shell
# .env
OPENAI_API_KEY=sk-...f8
```

Start ICE in its own terminal and leave it running:

```shell
scripts/run-local.sh
```

On the first run, downloading the Docker container will take a few minutes.

## Shell into the Docker container

Open a shell in the container and use it to run all commands in the upcoming chapters:

```shell
docker compose exec ice bash
```

This command gives you a shell in the `ice` directory. Any files you create under this directory will be visible in the container.
