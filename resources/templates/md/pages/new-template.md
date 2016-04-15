{:title "Creating a new template"
 :layout :page
 :page-index 2}

## Template - View

### Render a simple template

In this chapter we will create a new template to display the blog entries index page. Closp uses [Selmer](https://github.com/yogthos/Selmer) as templating engine which itself is inspired by the django templating engine.

In `resources/templates` create a new folder _blog_ and inside that folder a new file _index.html_. 
Open that file and insert the following content:

```
{% extends "templates/base.html" %}
{% block content %}
What an awesome blog.
{% endblock %}
```

`{% extends "templates/base.html" %}` means that your template extends the base template, which can be found and adapted in `resources/templates/base.html`.  
Your content then goes inside the `{% block content %}` content block.

To make use of your new template open the routes file defined in the previous chapter `src/clj/foo/example/routes/blog.clj`. Adapt the `index-page` function to look like this:

```
(defn index-page []
  (layout/render "blog/index.html"))
```

This requires a new import _layout_. Add the following to your _require_ section of your ns declaration:

```
[foo.example.layout :as layout]
```

If you switch to your browser now and reload the page <http://localhost:3000/blog> you will see 
the content of your index.html file wrapped in the base.html

![Index Template](/img/template-1.png)

### Add dynamic generation

To add dynamic data to your template you have two change two places. First the _blog.clj_ file and second your _index.html_, the both we just worked on.  
For now we will work with static data defined in the backend, we will use clojure datastructures for this. Add the following code to your _blog.clj_:

```
(def posts [{:title "First Post" :content "First Content"}
            {:title "Second Post" :content "Second Content"}])
```

And change your `index-page` function to look like this:

```
(defn index-page []
  (layout/render "blog/index.html" {:posts posts}))
```

The `render` function accepts a map as second parameter where we can pass in whatever data we want. For now we stick with our static posts var defined before.

Now open your _index.html_ file and change it to look like this:

```
{% extends "templates/base.html" %}
{% block content %}

<h1>Awesome Blog</h1>

{% for post in posts %}
<b>{{post.title}}</b>
<p>{{post.content}}</p>
{% endfor %}

{% endblock %}
```

If you refresh the page now it should look like this:  

![Index with posts](/img/template-2.png)

Thats it, you can now add dynamic content.