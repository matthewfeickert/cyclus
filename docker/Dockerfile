# Builds Cyclus in an environment of the Pixi workspace and makes an image in
# which that environment is activated by default. This follows the recipe of
# https://github.com/CHTC/templates-GPUs/tree/master/containers/pixi with the
# addition that Cyclus is built and its install directory is kept in the image.
#
# The image has to be built from the top level directory of the repository:
#
#   docker build -f docker/Dockerfile .

ARG ENVIRONMENT="default"

FROM ghcr.io/prefix-dev/pixi:noble AS build

# Redeclaring ARGS in a stage without a value inherits the global default
ARG ENVIRONMENT
# Extra arguments for install.py
ARG BUILD_FLAGS="--allow-milps --parallel"

WORKDIR /app
COPY . .
RUN pixi install --locked --environment $ENVIRONMENT
RUN pixi run --locked --environment $ENVIRONMENT build $BUILD_FLAGS
# Activate ENVIRONMENT at container runtime by using the Pixi environment
# activation script as the container ENTRYPOINT.
# Set fallback environmental variables to guard against batch system defaults
RUN echo "#!/usr/bin/env bash" > /app/entrypoint.sh && \
    echo 'export USER="${USER:-$(id -u)}"' >> /app/entrypoint.sh && \
    echo 'export LOGNAME="${LOGNAME:-$USER}"' >> /app/entrypoint.sh && \
    echo '[ -w "${HOME:-/}" ] || export HOME="$(mktemp -d)"' >> /app/entrypoint.sh && \
    pixi shell-hook --locked --environment $ENVIRONMENT --shell bash >> /app/entrypoint.sh && \
    echo 'exec "$@"' >> /app/entrypoint.sh && \
    chmod 0755 /app/entrypoint.sh

# Runs the tests, which are only in the 'test' environment, so this stage has
# to be built with '--build-arg ENVIRONMENT=test'. It is not part of the final
# image and is only built when asked for with '--target test'.
FROM build AS test

ARG ENVIRONMENT

RUN pixi run --locked --environment $ENVIRONMENT test

FROM ghcr.io/prefix-dev/pixi:noble AS final

ARG ENVIRONMENT

WORKDIR /app
COPY --from=build /app/.pixi/envs/$ENVIRONMENT /app/.pixi/envs/$ENVIRONMENT
COPY --from=build /app/pixi.toml /app/pixi.toml
COPY --from=build /app/pixi.lock /app/pixi.lock
# The ignore files are needed for 'pixi run' to work in the container
COPY --from=build /app/.pixi/.gitignore /app/.pixi/.gitignore
COPY --from=build /app/.pixi/.condapackageignore /app/.pixi/.condapackageignore
# Cyclus is installed into the install directory of the workspace and not into
# the Pixi environment (c.f. CYCLUS_INSTALL_PREFIX in pixi.toml)
COPY --from=build /app/install /app/install
COPY --from=build /app/entrypoint.sh /app/entrypoint.sh

# The ENTRYPOINT is not used for the RUN instructions of an image that is
# built FROM this one, so also make it the shell of those, for the environment
# to be activated while projects that depend on Cyclus are built.
SHELL [ "/app/entrypoint.sh", "/bin/bash", "-c" ]
ENTRYPOINT [ "/app/entrypoint.sh" ]
