{:title "Writing to database"
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
(defn create-page []
  (layout/render "blog/create.html"))

(defn create [title content]
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

And finally we will add the _create.html_ file in `resources/templates/blog` which contains a very basic form to add the title and the content:

```
{% extends "templates/base.html" %}
{% block content %}

<form action="/blog" method="post">
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

If you followed all the steps you should switch to your REPL now and run:

> (reload)

This is necessary because we added a route and for closp to pick it up we have to reload all namespaces.

When you are done the form should look like this:

![Create Post Form](/img/create-post-01.png)
