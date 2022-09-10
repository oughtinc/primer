# Before we start

This tutorial requires that you've set up the [Interactive Composition Explorer](https://github.com/oughtinc/ice) (ICE):

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) for the backend
2. Install Node via [nvm](https://github.com/nvm-sh/nvm) for the frontend
3.  Clone ICE:

    ```
    git clone https://github.com/oughtinc/ice9.git ice
    ```
4.  Install the required Node packages:

    ```
    (cd ice/ui && nvm use && npm install)
    ```
5.  Start ICE in its own terminal and leave it running:&#x20;

    ```
    scripts/run-local.sh
    ```

