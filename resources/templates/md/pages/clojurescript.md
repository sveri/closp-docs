{:title "Clojurescript"
 :layout :page
 :page-index 3000}

Closp comes with an example using clojurescript. You can find it at: `src/cljs/foo/example/core.cljs`.
It shows how to use datascript with reagent. I wont go deeper into the clojurescript example here, but instead will
focus on how to use cljs during development and production time

## Developing clojurescript

We make use of figwheel, so to get started, first open a command line and run:

    lein figwheel

This will start a process that transpile the cljs code to javascript and load it in the browser. It will also open a
websocket connection, so that code changes can be pushed to the browser. This happens automatically every time a
cljs or cljs file is being saved.

## Single Page Application or Multi Page

With closp you can go either route, it is by default set up to server different cljs files for different templates. To do
this some manual work needs to be done.

There are three important files here to look out for:

- `src/cljs/foo/example/core.cljs` or whatever cljs file you want to work an and which contains a `main` function
- `resources/templates/home/example.html` or whatever template you want to use to serve the javascript
- `env/dev/cljs/project_name/dev.cljs` This file defines which function should be loaded on every figwheel reload

These three define what you see during development.

In the template you refer to the function to call like this:

    {% block js-bottom %}
    <script type="text/javascript">foo.example.core.main();</script>
    {% endblock %}

And in `dev.cljs` you define the function to call during reload like this:

    (:require [foo.example.core :as core])
    (defn main [] (core/main))

where foo.example.core is the namespace containing your main function.

You can add as many clojurescript entry points for your application as you want. You just have to adjust the `dev.cljs`
file to load the main function that you are currently working on.

## Production time

When running in production the `dev.cljs` is not compiled into the resulting javascript. Instead, when running:

   lein rel-jar

the clojurescript will be transpiled to a minified javascript that is put into the resulting jar file and can be referred
by the templates.

This is done automatically by this line:

    <script src="/js/compiled/app.js"></script>

in the `resources/templates/base.html` file.