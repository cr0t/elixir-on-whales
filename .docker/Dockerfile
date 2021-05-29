ARG ERLANG_VERSION
ARG ELIXIR_VERSION

FROM hexpm/elixir:${ELIXIR_VERSION}-erlang-${ERLANG_VERSION}-alpine-3.13.3

# These vars need to stay here to avoid being cleared
ARG NODE_VERSION
ARG POSTGRES_VERSION

# Install dependencies
RUN apk add --no-cache --update --repository=http://dl-cdn.alpinelinux.org/alpine/v3.13/main \
    build-base git curl inotify-tools openssl ncurses-libs npm \
    nodejs~${NODE_VERSION} \
    postgresql-dev~${POSTGRES_VERSION}

ENV LANG=C.UTF-8

# Create a directory for the app code
WORKDIR /app

# Install Hex and Rebar
RUN mix local.hex --force && \
    mix local.rebar --force
