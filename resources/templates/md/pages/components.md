{:title "Components"
 :layout :page
 :page-index 1100}

In this chapter I will explain what components are and how to use them.

## What is it good for.

Components are abstractions that put a wrapper around a stateful resource to be able to pass them around. The original
implementation was from Stuart Sierra and there is also a talk about them on youtube from him, which is recommended
to be watcherd.
They also provide a way to implicitly declare a dependency tree among the statefule resources.


## Usage

Components implement the `Lifecycle` protocol which defines two methods:

    (start [comp] comp)
    (stop [comp] comp)

This is all you have to implement to make use of them. In the `start` function you start up your component in whatever
way you have to. Finally you add the resource to the component map by returning the map and associng your resource into.
To stop the component you can access the resource by calling get on the `stop` function parameter. Here you can do whatever
you need to shutdown the resource.

## Putting them together

To integrate a component into closp please open: ´src/clj/foo/example/components/components.clj`, require the appropriate
namespace and add it to the dev and prod components.
A dependency is implicitly declared like this:

    :config (c/new-config (c/prod-conf-or-dev))
    :db (component/using (new-db) [:config])

Here the `db` component requires the `config` component. This means that you can acces the configuration in the database
component. Example code looks like this:

    (defrecord Db [config]
      component/Lifecycle
      (start [component]
        (let [db-url (get-in config [:config :jdbc-url])]
          (defdb db db-url))
        component)
      (stop [component] component))

    (defn new-db []
      (map->Db {}))))

You call an accessor function that creates a new instance of the component. The record will be passed in the configuration
component which can then be accessed in the `start`and `stop` functions.

## Components that come with closp

Closp by default makes use of several components.

### Config

Will read the configuration from `closp.edn` and make it accessible everywhere it is needed.

### Db

The component that defines a connection to the database.

### Locale

The component that defines the internationalization properties that come with closp and can be extended by the user.

### Handler

The component that defines the routes and middlewares of closp.

### Server

The component that starts and stops the server while making use of the handler.
