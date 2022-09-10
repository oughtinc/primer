# Before we start

The recipes in this tutorial are implemented using the [Interactive Composition Explorer](https://github.com/oughtinc/ice) (ICE). If you'd like to follow along with the implementation (strongly recommended), set it up first.

## Prerequisites

1. Install [Docker](https://www.docker.com/products/docker-desktop/) for the backend
2. Install the NodeJS version manager [nvm](https://github.com/nvm-sh/nvm) for the frontend

## Installation

1.  Clone ICE:

    ```shell
    git clone https://github.com/oughtinc/ice9.git ice
    ```
2.  Install NodeJS and the required Node packages:

    ```shell
    (cd ice/ui && nvm install && nvm use && npm install)
    ```

## Run the ICE server

Start ICE in a separate terminal and leave it running:&#x20;

```shell
cd ice && scripts/run-local.sh
```

