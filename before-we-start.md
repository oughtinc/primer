# Before We Start

The recipes in this primer are implemented using the [Interactive Composition Explorer](https://github.com/oughtinc/ice) (ICE). If you'd like to follow along with the implementation (strongly recommended), set it up first.

## Prerequisites

1. Install [Docker](https://www.docker.com/products/docker-desktop/) for the backend
2. Install the NodeJS version manager [nvm](https://github.com/nvm-sh/nvm) for the frontend

## Installation

1.  Clone ICE:

    ```shell
    git clone https://github.com/oughtinc/ice.git
    ```
2.  Install NodeJS and the required Node packages:

    ```shell
    cd ice/ui
    nvm install
    nvm use
    npm install
    ```

## Run the ICE server

Start ICE in a separate terminal and leave it running:&#x20;

```shell
cd ice
scripts/run-local.sh
```

## Shell into the Docker container

Open a shell in the container to run all the commands in the upcoming chapters:

```shell
docker compose exec backend bash
```
