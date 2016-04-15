{:title "Create Post"
 :layout :page
 :page-index 500}

In the last chapter we read our data from the database. Now we will create a simple form to add new blog posts.

## Create Post

Open _blog.clj_ in `foo.example.db`. We will add two new functions to create and update posts:

```
(defn create-post [m]
  (let [data (-> (select-keys m [:title :content :draft])
                 (merge {:created_at (quot (System/currentTimeMillis) 1000)}))]
    (insert post (values data))))
```

Also you will have to update the require section of that namespace because of the new functions
that we just added: 

```
[korma.core :refer [select insert values]]
```

Next we have to update our routes file. Open _blog.clj_ in `foo.example.routes` and add the following code:

```
(defn create-post-page []
  (layout/render "blog/create-post.html"))

(defn create-post [title content]
  (b-db/create-post {:title title :content content})
  (resp/redirect "/blog"))
```

We also have to update the _require_ part of the routes namespace to include the _resp_ directive and _POST_ function:

```
(:require [compojure.core :refer [routes GET POST PUT]]
            [foo.example.db.blog :as b-db]
            [foo.example.layout :as layout]
            [ring.util.response :as resp])
```

Next update the routes function and add these two routes:


```
(GET "/post/create" [] (create-post-page))
(POST "/post" [title content] (create-post title content))
```

And finally we will add the _create-post.html_ file in `resources/templates/blog` which contains a very basic form to add the title and the content:

```
{% extends "templates/base.html" %}
{% block content %}

<form action="/post" method="post">
    <input name="__anti-forgery-token" type="hidden" value="{{af-token}}"/>

    <div class="form-group">
        <label for="title">Title</label>
        <p>
            <input class="form-control" id="title" name="title" />
        </p>
    </div>

    <div class="form-group">
        <label for="content">Content</label>
        <p>
            <textarea class="form-control" id="content" name="content"></textarea>
        </p>
    </div>

    <div class="form-group">
        <input class="btn btn-primary" type="submit" value="Create">
    </div>

</form>

{% endblock %}
```

Please note the hidden *__anti-forgery-token_* input field. It is needed for every POST / PUT / DELETE request
and protects you from cross site request forgery. It is automatically available in every selmer template that
you render with the _layout/render_ function.

If you followed all the steps you should switch to your REPL now and run:

> (reload)

This is necessary because we added a route and for closp to pick it up we have to reload all namespaces.

When you are done the form at: <http://localhost:3000/post/create> should look like this:

![Create Post Form](/img/create-post-01.png)
