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

    {:default {:jdbc-url "jdbc:postgresql://localhost:5432/foobar?user=foobar&password=topsecret"}

According `docker-compose.yaml` file:

```yaml
name: foobar

services:
  db:
    image: postgres:17.4-bookworm
    environment:
      POSTGRES_DB: foobar
      POSTGRES_USER: foobar
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

## Database

Create a new migration called _create users table_:

    (migratus.core/create (:db.sql/migrations state/system) "create-account-table")

This creates new timestamped files in `resources/migrations` with an `.up.sql`
and a `.down.sql` extension.

In `[timestamp]-create-account-table.up.sql`:

```sql
create table if not exists account (
    id serial primary key,
    name varchar(100) not null,
    password varchar(255) not null
);
```

In `[timestamp]-create-account-table.down.sql`:

```sql
drop table if exists account;
```

Run the migration:

    (migratus.core/migrate (:db.sql/migrations state/system))

Rollback the migration:

    (migratus.core/rollback (:db.sql/migrations state/system))

Rollback all migrations:

    (migratus.core/migrate (:db.sql/migrations state/system))

Create queries in `resources/sql`, e.g. `queries.sql`:

```sql
-- :name create-account! :<!
-- :doc inserts and returns an account
insert into account (name, password)
values (:name, :password)
returning *;

-- :name get-account-by-id :? :1
-- :doc gets a single account given its id
select *
from account 
where id = :id;

-- :name list-accounts
-- :doc lists all accounts
select *
from account;
```

Create a user:

    (reset)
    (go) ;; TODO: really needed?
    ((:db.sql/query-fn state/system) :create-account! {:name "Joe" :password "Secret"})
    ((:db.sql/query-fn state/system) :get-account-by-id {:id 1})
    ((:db.sql/query-fn state/system) :list-accounts {})

Define a shortcut in `user.clj`:

    (comment
      (go)
      (reset)
      (def query-fn (:db.sql/query-fn state/system)))

Evaluate with `[Ctrl]`-`[Enter]`, then use it as follows in the REPL:

    (query-fn :create-account! {:name "Jill" :password "Topsecret"})

## Routing

Create a new controller under `src/clj/com/github/patrickbucher/foobar/web/controllers/hello.clj`:

```clojure
(ns com.github.patrickbucher.foobar.web.controllers.health
  (:require
    [ring.util.http-response :as http-response]))

(defn hello! [req]
  (http-response/ok {:hello "World"}))
```

Register it in `src/clj/com/github/patrickbucher/foobar/web/routes/api.clj`:

```clojure
(defn api-routes [_opts]
   [ ;; ...
   ["/hello" {:get hello/hello!}]])
```