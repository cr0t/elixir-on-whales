# Elixir on Whales

> Inspired by [Dockerizing Ruby and Rails development] blog post.

Docker Compose template that helps in bootstrapping an environment for Elixir and/or Phoenix applications.

Below we provide a step-by-step instruction on how to bootstrap your local development environment.

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

root@f253eaa0b3e7:/app# mix archive.install hex phx_new 1.5.8
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

As you see, we made a trick with `String.replace("app_dev", "app_test")` in `test.exs` configuration file - this is a dirty workaround that converts DATABASE_URL to its own database url to run tests.

> Do not forget to explicitly run `MIX_ENV=test mix test` command.

Ok! Now you can try to open http://localhost:4000/ in the browser on the host machine and check if you see Phoenix welcome page.

[Dockerizing Ruby and Rails development]: https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development

## Using `:observer` in macOS

If you use macOS and want to be able to access `:observer` GUI, there is a workaround we found [here](https://github.com/moby/moby/issues/8710).

First, you need to install a couple of utilities:

```bash
$ brew install socat
$ brew cask install xquartz
```

> Probably, you will need to restart your system.

[`socat`](http://www.dest-unreach.org/socat/) ("SOcket CAT: netcat on steroids") is a small utility which can help us with proxying ports.

The [XQuartz project](https://www.xquartz.org/) is an open-source effort to develop a version of the X.Org X Window System that runs on OS X.

Then you also need to obtain your host machine local network IP address (try to run `ipconfig getifaddr en0` or look into System Preferences). You need to update `.env` file with this piece of information (see `X_WINDOWS_ADDRESS`).

Now we need to start proxy on the host system (use a separate terminal to run this command):

```bash
$ socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"
```

After having this done we can start using magic of remote X Window sessions:

```bash
$ docker-compose exec phoenix iex
Erlang/OTP 22 [erts-10.6.2] [source] [64-bit] [smp:6:6] [ds:6:6:10] [async-threads:1] [hipe]

Interactive Elixir (1.9.4) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> :observer.start()
:ok
```

Now you should see something similar to this on your screen:

![docker-observer-mac-os-x-windows](https://user-images.githubusercontent.com/113878/73979910-6b420200-492f-11ea-9b1d-d526b11c9d06.png)
