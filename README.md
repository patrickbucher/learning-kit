# Setup

Workshop: [yogthos/kit-workshop](https://github.com/yogthos/kit-workshop)

## Prerequisites

- Clojure

Install `clj-new`:

    clojure -Ttools install-latest :lib com.github.seancorfield/clj-new :as clj-new

Create a new project:

    clojure -Tclj-new create :template io.github.kit-clj :name com.github.patrickbucher/foobar

Start a REPL:

    cd foobar
    clojure -M:dev:nrepl

Start the application from the REPL:

    (go)

The Swagger API documentation becomes available at [localhost:3000/api](http://localhost:3000/api).

## Modules

Pull the modules form the remote repository (see `:modules` in `kit.edn`):

    (kit/sync-modules)

List available modules:

    (kit/list-modules)

Install a module with flags, e.g. `:kit/sql` for PostgreSQL:

    (kit/install-module :kit/sql {:feature-flag :postgres})

Configure the database credentials in `resources/system.edn`,
e.g. (dropping `:dev` and `:test` in favour of `:default`):

    {:default {:jdbc-url "jdbc:postgresql://localhost:5432/kit-hello-world?user=kit-hello-world&password=topsecret"}

According `docker-compose.yaml` file:

```yaml
name: kit-hello-world

services:
  db:
    image: postgres:17.4-bookworm
    environment:
      POSTGRES_DB: kit-hello-world
      POSTGRES_USER: kit-hello-world
      POSTGRES_PASSWORD: topsecret
    ports:
      - '5432:5432'
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

Reload the system (e.g. re-connects the database with new credentials):

    (reset)
