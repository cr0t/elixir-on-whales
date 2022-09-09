# Elixir on Whales

> Inspired by [Dockerizing Ruby and Rails development] blog post.

This repository provides a template that supposed to help to organize an development environment for Elixir and/or Phoenix applications.

Below there is step-by-step instructions on how to bootstrap a new Phoenix application.

You can also find a few tips on how to use `:observer` GUI from inside a running Docker container.

## Phoenix Application

```console
$ git clone git@github.com:cr0t/elixir-on-whales.git <your-app-name>
$ cd <your-app-name> && rm -rf .git
$ mv .env.example .env
$ mv README.md README-ON-WHALES.md # we need this to avoid conflicts with app's README.md
$ mv LICENSE LICENSE-ON-WHALES     # and with app's license
```

Now you can open `docker-compose.yml`, remove services you do not need and adjust versions and other settings in the `.env` file before you proceed.

```console
$ docker-compose up
```

This will download Docker images and set up containers for us. Don't exit (yet)!

When this is done, **in a separate terminal** you need to start a new shell session to install Phoenix framework inside the container.

```console
$ docker-compose exec shell sh
/app# mix archive.install hex phx_new 1.6.12
/app# mix phx.new . --module WhalesExample
```

> Replace `WhalesExample` with the name for your application.

Now you should edit `config/dev.exs` and `config/test.exs` files. You can do it inside the shell session, or via editor on your host machine.

You need to ensure that your application will use correct database URL, and server will run on 0.0.0.0 (by default, Phoenix starts a server on 127.0.0.1 interface, so it makes it unreachable outside of container).

Below there configuration files (not full, only sections that need to be changed).

> Please, ensure that you replaced `WhalesExampleWeb` with your application module name in the examples below.

`config/dev.exs` highlights:

```elixir
config :app, WhalesExampleWeb.Repo,
  url: System.get_env("DATABASE_URL") || "postgres://postgres:secret@postgres:5432/app_dev",
  stacktrace: true,
  show_sensitive_data_on_connection_error: true,
  pool_size: 10

# ...

config :app, WhalesExampleWeb.Endpoint,
  http: [ip: {0, 0, 0, 0}, port: 4000],
```

`config/test.exs` highlights:

```elixir
database_url =
  System.get_env("DATABASE_URL") || "postgres://postgres:secret@postgres:5432/app_dev"

config :app, WhalesExampleWeb.Repo,
  url: database_url |> String.replace("app_dev", "app_test#{System.get_env("MIX_TEST_PARTITION")}")
  pool: Ecto.Adapters.SQL.Sandbox
```

Now we can create database, restart Docker containers and get Phoenix applpication up and running!

```console
/app# mix ecto.create
/app# exit
$ docker-compose stop # or down
$ docker-compose start # or up
```

> We need to run two last commands to re-start all services defined in `docker-compose.yml` (if we left it original).

As you see, we made a trick with `String.replace("app_dev", ...)` in `test.exs` configuration file - this is a dirty workaround that converts `DATABASE_URL` environment variable to its own database url to run tests.

> Do not forget to explicitly run `MIX_ENV=test mix test` command.

Ok! Now you can try to open http://localhost:4000/ in the browser on the host machine and check if you see Phoenix welcome page.

<table>
  <tr>
    <td><img width="1378" alt="image" src="https://user-images.githubusercontent.com/113878/189381540-651b1ac1-f637-40dc-bba9-c21dabb15644.png"></td>
    <td><img width="1378" alt="image" src="https://user-images.githubusercontent.com/113878/189381588-18c4ca17-3d25-445b-ad2e-90ed8e07690d.png"></td>
  </tr>
</table>

[Dockerizing Ruby and Rails development]: https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development

## Using `:observer` in macOS

If you use macOS and want to be able to access `:observer` GUI, there is a workaround we found [here](https://github.com/moby/moby/issues/8710).

First, you need to install a couple of utilities:

```console
$ brew install socat
$ brew cask install xquartz
```

> Probably, you will need to restart your system.

[`socat`](http://www.dest-unreach.org/socat/) ("SOcket CAT: netcat on steroids") is a small utility which can help us with proxying ports.

The [XQuartz project](https://www.xquartz.org/) is an open-source effort to develop a version of the X.Org X Window System that runs on OS X.

Then you also need to obtain your host machine local network IP address (try to run `ipconfig getifaddr en0` or look into System Preferences). You need to update `.env` file with this piece of information (see `X_WINDOWS_ADDRESS`).

Now we need to start proxy on the host system (use a separate terminal to run this command):

```console
$ socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"
```

After having this done we can start using magic of remote X Window sessions:

```console
$ docker-compose exec phoenix iex
Erlang/OTP 22 [erts-10.6.2] [source] [64-bit] [smp:6:6] [ds:6:6:10] [async-threads:1] [hipe]

Interactive Elixir (1.9.4) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> :observer.start()
:ok
```

Now you should see something similar to this on your screen:

![docker-observer-mac-os-x-windows](https://user-images.githubusercontent.com/113878/73979910-6b420200-492f-11ea-9b1d-d526b11c9d06.png)
