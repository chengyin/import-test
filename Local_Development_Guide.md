# Local Development Guide [![CircleCI](https://circleci.com/gh/slab/slab.svg?style=svg&circle-token=c2942e586d594f8e1aa79d2be4e947b38c7775ea)](https://circleci.com/gh/slab/slab)

Our tech stack is React + Elixir/Phoenix + Postgres + Docker. Most of the team uses Visual Studio Code on Mac but other setups are supported.

![](https://static.slab.com/prod/uploads/posts/images/MTeHl0cfxO2Lx1Tkx3YHvbEK.jpg)

# Setup

## Installation

Clone the repo and modify the development config.

```
git clone git@github.com:slab/slab.git
cd slab
cp config/dev.example.exs config/dev.exs
vim config/dev.exs
```

In the `dev.exs` file you just copied, change the name from &quot;jason&quot; to yours. Your local application will run with this prefix as the subdomain ex. https://jason.slabdev.com/.

## Configure Docker

Download and install [Docker](https://docs.docker.com/docker-for-mac/). You will then need access to Slab's DockerHub account after which you can build the server images and run them. This might take a few minutes.

```
docker login
docker-compose build
docker-compose run slab mix ecto.setup —rm
```

The database should now be set up with some seed data.

# Running Locally

To run Slab locally:

```
docker-compose up -d
docker-compose logs --tail=25 -f web
```

Once the webpack green text appears, the build is complete and watching for changes.

You should now be able to visit `https://{prefix}.slabdev.com` (which points to 127.0.0.1 via AWS) to see the app running locally.

_Tip: Many useful command shortcuts (some not covered in this guide) are listed in `package.json`. For example `npm run docker:up` will run the above two docker-compose commands._

## Logging in

The users are seeded with Game of Thrones characters. They all share the same password. The major characters Daenarys, Tyrion, Cersei, Jamie, Sansa, &amp; Jon are admins.

```
Team: {prefix}.slabdev.com
Username: sansa@slab.com
Password: password
```

## Hot reloading

You won&#39;t have to restart Slab from the server too often. Front end changes are watched by Webpack and will automatically rebuild and refresh your browser to load the new changes. Phoenix will also automatically rebuild as needed but is only configured to refresh the browser on views and templates changes (which we do not use very much anymore).

## Connecting to a specific server

Our dev environment runs two Phoenix servers (to reflect clustering in prod) and uses Nginx to load balance between them. If you want your browser to connect to a specific server, create a cookie called `slab_upstream` with a value of either `slab_1` or `slab_2`. You can also inspect the IP of the server that served any request by checking the `X-Slab-Server` header.

## Exposing Remotely

Sometimes it useful to make your local development environment accessible from the public internet, for example to test integration webhooks or realtime collaboration. To do so:

1. Ask to get an ngrok account created and tunnel configured for you.
1. Add `127.0.0.1 {prefix}.slabdev.com` and `::1 {prefix}.slabdev.com` to your `/etc/hosts` file.
1. Download [ngrok](https://ngrok.com/download).
1. Unzip and move binary to a local executable path ex. `sudo mv ~/Downloads/ngrok /usr/local/bin/`
1. Log in to ngrok and find your auth token under the [auth tab](https://dashboard.ngrok.com/auth), and configure: `ngrok authtoken YOURAUTHTOKEN`
1. Edit your local configuration file in `~/.ngrok2/ngrok.yml` to the following:

```yml
authtoken: YOURAUTHTOKEN
tunnels:
  slabdev:
    proto: http
    addr: 80
    hostname: {prefix}.slabdev.com
```

Now to expose your development environment just run `ngrok start slabdev` or `npm run ngrok`. You can use ngrok's [web interface](http://127.0.0.1:4040/) to inspect and disagnose many issues. Note that the change to your `/etc/hosts` file will cause local traffic to go directly to your local server, not through ngrok (which means ngrok does not need to be running for normal local development).

# Development Tooling

![](https://static.slab.com/prod/uploads/posts/images/_E9iJMIALEII-voy4t6u8E_a.jpg)

## Prerequisites

Since some of these tools run on the host machine, Slab dependencies will need to be installed there to use these tools. This means installing Node and NPM (recommend using nvm) for front end linting and Elixir and Rust (on Macs, recommend using Homebrew) for back end linting. After this you can just do `npm install` and `mix deps.get` to install the respective front and back end dependencies.

## Linters

We use ESLint and sass-lint for frontend and Credo for backend. All can run on the Docker guest container:

* npm run lint:ex - Back end Elixir
* npm run lint:gql - GraphQL
* npm run lint:js - Front end Javascript
* npm run lint:sass - Front end sass
* npm run lint - Run all linters

If you followed the previous prerequisites section, you can also run the above commands in the host container.

## Code Formatters

Slab uses formatters to automatically enforce consistent and maintainable code on the front and back end.

For JavaScript, We use [Prettier](https://github.com/prettier/prettier) for basic formatting. Certain linter violations are auto fixable. For Elixir, we use the language native formatter. For scss, there is no formatter available. sass-lint-auto-provides auto fixes for linter violations. Commands for formatting:

* npm run format:ex - Elixir
* npm run format:js - JavaScript through ESLint/TSLint auto fix
* npm run format:sass - sass through sass-lint-auto-fix
* npm run format - Run all formatters

## Editor Notes

It&#39;s a good idea to integrate linting and formatting into your editor so you can have them automatically run during development.

**Visual Studio Code**

Install vscode-elixir, Sass, ESLint, Prettier - Code formatter, ElixirLS: Elixir support and debugger packages. You can install them all with the CLI:

```
$ code --install-extension dbaeumer.vscode-eslint esbenp.prettier-vscode glen-84.sass-lint JakeBecker.elixir-ls mjmcloug.vscode-elixir robinbentley.sass-indented
```

Our TS/JS linter enforces imports ordering. VSCode's "organize imports" command (Option+Shift+O) is useful for automatically sort imports.

**Sublime Text 3**

1. Install the SublimeLinter-eslint package and follow the instructions (including installing eslint globally via npm)
1. Install the Babel, JsPrettier and Elixir packages
1. Open a Javascript file and then _View &gt; Syntax &gt; Open all with current extensions as... &gt; Babel &gt; Javascript (Babel)_

## Storybook

You can view our standardized UI components through storybook. Storybook is hosted at [https://storybook.slabdev.com](https://storybook.slabdev.com/)

To write stories for UI components, please read the [+Storybook Guide](https://the.slab.com/posts/d74f75d5).

## Postico

It is recommended you use [**Postico**](https://eggerapps.at/postico/) or some other database GUI to inspect and manage Slab data. Use the following configuration to connect to the local database.

* username: slab
* password: slab
* host: localhost
* port: 15432

# Doing Things

## Restarting Phoenix

If you get the "could not compile application: slab" error or see "You must restart your server after changing the following config or lib files" in logs you can recompile the Phoenix app by running `npm run docker:compile` (you don't have to terminate the BEAM VM).

## Elixir REPL

To just use Elixir&#39;s REPL you can just run `iex` and can now run Elixir commands and test their output. More useful however is loading Slab into `iex` so you can run Slab commands too. This will run the REPL with the application libraries and contexts loaded:

* Bash into docker with `npm run docker:ssh`.
* Once you are in, run `iex -S mix phx.server`
* You can also run `iex -S mix` to skip starting the server (allowing multiple iex sessions)

You can now query for models like...

```
iex> Slab.User |> Ecto.Query.first |> Slab.GlobalRepo.one
```

You can also run imports and aliases inside the REPL to avoid repetitive typing:

```
iex> import Ecto.Query
iex> alias Slab.{Repo, User}
iex> query = from u in User, where: u.id == "abc123"
iex> Repo.one(query)
```

When code changes and are rebuilt, they are automatically loaded into `iex` so you do not have to quit and re-run (usually—sometimes this will error). Again back end changes usually do not trigger rebuilds so you will either have to refresh your browser that is on a Slab page (this will cause execution of back end code which the Phoenix will see is old and trigger a recompile) or in the REPL run `recompile`.

Note the REPL and Phoenix also share the same IO, it can be annoying when Webpack messages or cron jobs outputs here as well.

## Logging

While debugging, `IO.inspect` is your friend. It prints and returns the contents of anything it is passed. Because of the return it can be easily inserted in chains:

```
Slab.User
|> IO.inspect
|> Ecto.Query.first
|> IO.inspect
|> Slab.GlobalRepo.one
|> IO.inspect
```

## Testing

Just ssh into Docker to run our tests.

```
npm run docker:ssh
npm run test
```

You can also run `npm run test:ex` or `npm run test:js` to only run front or back end tests.

Code coverage is also available:

```
npm run docker:ssh
MIX_ENV=test mix coveralls.html
open .coverage/excoveralls.html
```

## Recreating the database

If you pull and a breakage seems schema related, try to drop and rebuild the DB. To recreate the database and start over with fresh data:

```
npm run docker:ssh
mix ecto.reset
```

## Running Migrations

We use Ecto as the database wrapper (apparently they do not want to be called an ORM), which also provides other useful mix commands:

* `mix ecto.gen.migration migration_name` - Creates a new migration
* `mix ecto.migrate` - Runs all new migrations
* `mix ecto.rollback` - Rolls back one migration

## Installing a new package

After running the commands below, make sure to change `package.json` s.t. each dependency references a specific version number.

```
npm install <foo_new_package> —-save
npm run docker:rebuild
```

# Directory guide

Almost all the interesting application code is in the *lib/slab* directory. All of the interesting front end code is in *assets/*. The structure is following Phoenix 1.3&#39;s Bounded Contexts design.

```
assets
  css            - Front-end Sass source
  js             - Front-end Javascript source
bin              - Scripts for Docker, building, deploying
config           - Configuration files
deps             - Phoenix installed dependencies
docker           - Dockerfiles
lib
  absinthe       - Slab specific Absinthe code
  plug           - Slab specific plugs
  postgrex       - Define ltree/lquery for postgres
  slab
    [feature]    - Top level Slab feature
    tandem       - Realtime collaboration
    web          - Back end channels, controllers and helpers
node_modules     - Javascript installed dependencies
priv
  gettext        - Internationalization, if we ever get to that
  repo           - Migrations and seed scripts
  static         - Webpack build output
rel              - Config and output location for release builds
test
  slab           - Roughly mirrors lib/slab for testing
```

## Frontend Guidelines

Organize files in the `assets/js` with these rules:

* Is the file only used by a top-level features? If yes, start with `assets/js/:feature_name` folder
  * Sub-folders are allowed: `js/:feature_name/:sub_feature_name`. A use case is when a page have multiple tabs, each tab could be a sub feature
* If the file is shared between top-level features, put it in the shared top level folder: `assets/js/{grapql, component, shape, utils}`
* All the application entry point files are in `js/app`
