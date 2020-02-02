# Elixir on Whales

> Inspired by https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development article.

It's a Docker Compose template repository that should helps in bootstrapping an environment for writing Elixir and/or Phoenix applications.

Below we provided a step-by-step instructions on how to bootstrap your local development environment.

## New Phoenix Application

```bash
$ git clone git@github.com:cr0t/elixir-on-whales.git <your-app-name>

$ cd <your-app-name> && rm -rf .git

$ mv .env.example .env

$ mv README.md README-ON-WHALES.md # we need this to avoid conflicts with app's README.md

# now you can open docker-compose.yml, remove services you do not need
# and adjust versions and other settings in the .env file before you proceed

$ docker-compose up
```

**In a separate terminal** you need to start a new shell session to install Phoenix.

```bash
$ docker-compose exec shell bash

root@f253eaa0b3e7:/app# mix archive.install hex phx_new 1.4.11
root@f253eaa0b3e7:/app# mix phx.new . --module <YourProjectName>
root@f253eaa0b3e7:/app# vim config/dev.exs
root@f253eaa0b3e7:/app# cat config/dev.exs
use Mix.Config

# Configure your database
config :app, ElixirTest.Repo,
  url: System.get_env("DATABASE_URL"),
  show_sensitive_data_on_connection_error: true,
  pool_size: 10

# For development, ...
root@f253eaa0b3e7:/app# cat config/test.exs
use Mix.Config

database_url =
  System.get_env("DATABASE_URL") || "postgres://postgres:secret@postgres:5432/app_dev"

# Configure your database
config :app, Rumbl.Repo,
  url: database_url |> String.replace("app_dev", "app_test"),
  pool: Ecto.Adapters.SQL.Sandbox

# We don't run a server during test...
root@f253eaa0b3e7:/app# mix ecto.create
root@f253eaa0b3e7:/app# exit
$ docker-compose stop # or down
$ docker-compose start # or up
```

> We need to run two last commands to re-start all services defined in `docker-compose.yml` (if we left it original).

Also, as you see, we did a hack with `String.replace("app_dev", "app_test")` in `test.exs` configuration file - this a dirty workaround that uses the same DATABASE_URL, but its own database to run tests.

> Do not forget to explicitly run `MIX_ENV=test mix test` command.

Ok! Now you can try to open http://localhost:4000/ in the browser on the host machine and check if you see Phoenix welcome page.
