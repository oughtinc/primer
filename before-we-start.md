# Before we start

This tutorial requires that you've set up the [Interactive Composition Explorer](https://github.com/oughtinc/ice) (ICE).

### Prerequisites

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) for the backend
2. Install Node via [nvm](https://github.com/nvm-sh/nvm) for the frontend

### Installation

1.  Clone ICE:

    ```
    git clone https://github.com/oughtinc/ice9.git ice
    ```
2.  Install the required Node packages:

    ```
    (cd ice/ui && nvm use && npm install)
    ```

### Run ICE server

1.  Start ICE in its own terminal and leave it running:&#x20;

    ```
    scripts/run-local.sh
    ```

