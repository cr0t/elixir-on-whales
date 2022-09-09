ARG ELIXIR_VERSION
ARG ERLANG_VERSION
ARG ALPINE_VERSION
ARG NODE_VERSION
ARG POSTGRES_VERSION

FROM hexpm/elixir:${ELIXIR_VERSION}-erlang-${ERLANG_VERSION}-alpine-${ALPINE_VERSION}

ENV LANG=C.UTF-8

# Install dependencies
RUN apk add --no-cache --update \
    build-base git curl vim inotify-tools openssl ncurses-libs npm \
    nodejs~${NODE_VERSION} \
    postgresql-dev~${POSTGRES_VERSION}

# Create a directory for the app code
WORKDIR /app

# Install Hex and Rebar
RUN mix local.hex --force && \
    mix local.rebar --force
