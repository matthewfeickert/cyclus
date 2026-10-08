The `Dockerfile` builds Cyclus in an environment of the [Pixi](https://pixi.sh/) workspace (`pixi.toml`, `pixi.lock`) and makes an image in which that environment is activated by default. It follows the [Pixi container recipe of CHTC](https://github.com/CHTC/templates-GPUs/tree/master/containers/pixi), with the addition that Cyclus is built and its install directory is kept in the image.

The image must be built from the top level directory of the Cyclus code space.

To build the image with the default environment of the workspace:

`docker build -f docker/Dockerfile .`

To build the image with the `test` environment, which also has what is needed to run the tests:

`docker build --build-arg ENVIRONMENT=test -f docker/Dockerfile .`

To build Cyclus and run its tests without making the final image:

`docker build --build-arg ENVIRONMENT=test --target test -f docker/Dockerfile .`

The arguments that are given to `install.py` can be changed with the `BUILD_FLAGS` build argument, which is `--allow-milps --parallel` by default.

The environment is activated by the entrypoint of the image, so commands run in it directly:

`docker run --rm <image> cyclus --version`

The entrypoint is also the shell of the image, so the `RUN` instructions of a Dockerfile that is built `FROM` the image are run in the environment as well.
