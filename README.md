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
    clj -M:dev:nrepl

Start the application from the REPL:

    (go)
