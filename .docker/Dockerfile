ARG ELIXIR_VERSION
ARG ERLANG_VERSION
ARG ALPINE_VERSION

FROM hexpm/elixir:${ELIXIR_VERSION}-erlang-${ERLANG_VERSION}-alpine-${ALPINE_VERSION}

# These two args need to stay here – otherwise they will be empty at RUN stage
ARG NODE_VERSION
ARG POSTGRES_VERSION

ENV LANG=C.UTF-8

# Install dependencies
RUN apk add --no-cache --update \
    build-base git curl vim inotify-tools openssl ncurses-libs npm \
    nodejs-current~${NODE_VERSION} \
    postgresql14-dev~${POSTGRES_VERSION}

# Create a directory for the app code
WORKDIR /app

# Install Hex and Rebar
RUN mix local.hex --force && \
    mix local.rebar --force
