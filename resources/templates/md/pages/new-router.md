{:title "Creating a new router"
 :layout :page
 :page-index 1}

## Router - Controller

Closp uses [compojure](https://github.com/weavejester/compojure) as its routing library.  
We will create now something that acts as the controller according to the MVC principle. To do so add a new file in 
`src/clj/foo/example/routes` named _blog.clj_.

### Namespace declaration
Inside _blog.clj_ add the follwing lines:

```
(ns foo.example.routes.blog
  (:require [compojure.core :refer [routes GET]]
            [ring.util.response :refer [response]]))
```

This creates a new namespace and requires the utilities needed to generate templates.  

### Routes definition
Then we will create a routes definition and we start with a simple `GET` request:

```
(defn blog-routes []
  (routes
    (GET "/blog" [] (index-page))))
```

The routes definition defines an `/blog` uri that will return our blog index page.  
- GET - the first statement defines the type of the route (POST / PUT / DELETE / ...)
- "/blog" - the second statement defines the URI of the rout
- [] - the third statement defines the parameters of that request, which are empty in this case
- (index-page) - and the last statement is the function to execute when a requset on "/blog" comes in.

### Render function

Finally we add a function that returns something. To do so add the following code above the routes definition:

```
(defn index-page []
  (response {:foo :bar}))
```

This will return a transit formatted message which I will explain later.

### Add routes to handler

First you will need to import your routes file in the handler. Open `src/clj/foo/example/components/handler.clj`. Add a new require statement there: 

```
[foo.example.routes.blog :refer [blog-routes]]
```

now search for the `get-handler` function and the comment: _;; add your application routes here_.  
It will look similar to this:

```
(defn get-handler [config locale]
  (-> (app-handler
        (into [] (concat (when (:registration-allowed? config) [(registration-routes config)])
                         ;; add your application routes here
                         [(cc-routes config) home-routes (user-routes config) base-routes]))
        ;; add custom middleware here

```

Add the `blog-routes` function:

```
[(blog-routes) (cc-routes config) home-routes (user-routes config) base-routes]
```

Finally you have to reset the web server to notify him of the new routes. To do so switch to the open REPL and execute:

```
(reset)
```

This will reload all namespace. Now browse to <http://localhost:3000/blog> and you will see the following output: 
> ["^ ","~:foo","~:bar"]

Thats it, you just created a new router and returned a request.