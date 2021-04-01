ARG ELIXIR_VERSION
FROM elixir:$ELIXIR_VERSION-slim

ARG NODE_MAJOR
ARG POSTGRES_MAJOR

# Common dependencies
RUN apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
  ca-certificates \
  inotify-tools \
  build-essential \
  gnupg2 \
  curl \
  less \
  git \
  && apt-get clean \
  && rm -rf /var/cache/apt/archives/* \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

# Add PostgreSQL to sources list
RUN curl -sSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
  && echo 'deb http://apt.postgresql.org/pub/repos/apt/ stretch-pgdg main' $POSTGRES_MAJOR > /etc/apt/sources.list.d/pgdg.list

# Add NodeJS to sources list
RUN curl -sL https://deb.nodesource.com/setup_$NODE_MAJOR.x | bash -

# Install dependencies
COPY .docker/Aptfile /tmp/Aptfile
RUN apt-get update -qq \
  && DEBIAN_FRONTEND=noninteractive apt-get -yq dist-upgrade \
  && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
  libpq-dev \
  postgresql-client-$POSTGRES_MAJOR \
  nodejs \
  $(cat /tmp/Aptfile | xargs) \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

ENV LANG=C.UTF-8

# Install Hex and Rebar
RUN mix local.hex --force \
  && mix local.rebar --force

# Create a directory for the app code
RUN mkdir -p /app

WORKDIR /app
