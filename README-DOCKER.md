# Phoenix on Whales

It's an attempt to write a Docker Compose template for the environment that helps us writing Elixir and/or Phoenix applications.

## New Phoenix Application

```bash
$ git clone git@github.com:cr0t/elixir-on-whales.git <your-app-name>

$ cd <your-app-name> && rm -rf .git

$ mv .env.example .env

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
root@f253eaa0b3e7:/app# mix ecto.create
root@f253eaa0b3e7:/app# exit
$ docker-compose stop # or down
$ docker-compose start # or up
```

> We need to run two last commands to re-start all services defined in `docker-compose.yml` (if we left it original).

Ok! Now you can try to open http://localhost:4000/ in the browser on the host machine and check if you see Phoenix welcome page.
