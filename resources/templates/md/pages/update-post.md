{:title "Update Post"
 :layout :page
 :page-index 600}

In this chapter we will add functions to edit existing posts.

Open _blog.clj_ in `foo.example.db`. We will add two new functions to update posts:

```
(defn update-post [id m]
  (let [data (select-keys m [:title :content :draft])]
    (update post (set-fields data) (where {:id id}))))

(defn get-by-id [id]
  (first (select post (where {:id id}))))
```

_update-post_ takes a second parameter _id_ which is the id of the actual post.

Next we have to edit our routes file. Open _blog.clj_ in `foo.example.routes` and add the following code:

```
(defn edit-post-page [id]
  (layout/render "/blog/edit-post.html" {:post (b-db/get-by-id id)}))

(defn edit-post [id title content]
  (b-db/update-post id {:title title :content content})
  (resp/redirect "/blog"))
```

Again we will have to add two more routes:

```
(GET "/post/edit/:id" [id] (edit-post-page id))
(POST "/post/edit" [id title content] (edit-post id title content))
```

And finally we create a new html template in `resources/templates` called _edit-post.html_ with the following content:

```
{% extends "templates/base.html" %}
{% block content %}

<form action="/post/edit" method="post">
    <input name="__anti-forgery-token" type="hidden" value="{{af-token}}"/>
    <input name="id" type="hidden" value="{{post.id}}"/>

    <div class="form-group">
        <label for="title">Title</label>
        <p>
            <input class="form-control" id="title" name="title" value="{{post.title}}"/>
        </p>
    </div>

    <div class="form-group">
        <label for="content">Content</label>
        <p>
            <textarea class="form-control" id="content" name="content">{{post.content}}</textarea>
        </p>
    </div>

    <div class="form-group">
        <input class="btn btn-primary" type="submit" value="Update">
    </div>

</form>

{% endblock %}
```

It is very similar to the _create_ template and with some switches you could just use one template for both.

Again run:

> (reload)

in the REPL to notify closp of the changed routes.

To be able to actually edit anything we will also adapt the _index.html_ file to add links to the blog post:

```
<b><a href="/post/edit/{{post.id}}">{{post.title}}</a></b>
```

which before was:

```
<b>{{post.title}}</b>
```