{:title "Creating a new model"
 :layout :page
 :page-index 4}

## Model

In this chapter we will create a new model for blogs and load the posts data from database.

### Migrations File

Migrations are defined as sql statements and stored in: `resources/sqlite`. You can find the ones for user there.  
Create two new files _2016-04-15-01-post.up.sql_ and _2016-04-15-01-post.down.sql_ there.  
Add the following content to _2016-04-15-01-post.up.sql_:

```
CREATE TABLE post (
id INTEGER CONSTRAINT PK_USERRC PRIMARY KEY AUTOINCREMENT NOT NULL,
title VARCHAR(30),
content VARCHAR(30),
created_at time,
draft BOOLEAN DEFAULT 0 NOT NULL);
--;;
INSERT INTO post ("title", "content", "draft") VALUES
('First Post', 'First Content', 0),
('Second Post', 'Second Content', 0),
('Third Post', 'Third Content', 0);
```

It will create a new table post with the columns
- id - Which is an auto generated primary key
- title - The title of the post
- content - The content of the post
- created_at - The time at which the post was created.

And it also contains three posts which are seeded into the database.

The _2016-04-15-01-post.down.sql_ file contains the DROP statement:

```
DROP TABLE post;
```

Now switch to your command line in your blog directory and execute:

> lein migrate

The output should look like this:

> Migrating #ragtime.jdbc.SqlDatabase{:db-spec {:connection-uri jdbc:sqlite:./db/blog.sqlite}, :migrations-table ragtime_migrations, :url jdbc:sqlite:./db/blog.sqlite, :connection-uri jdbc:sqlite:./db/blog.sqlite, :db {:type :sql, :url jdbc:sqlite:./db/blog.sqlite}, :migrator resources/migrators/sqlite, :seed nil}  
> Applying 2016-04-15-01.post 

### Database Namespace

Now to make use of the new database table we will create some clojure code to read from it. Closp uses [korma](http://sqlkorma.com/) for database access.

First open the _entities.clj_ file in `src/clj/foo/example/db`. Add two new vars:

```
(declare post)
(defentity post)
```
This will map the _post_ table to a korma entity called _post_.

Then create a new file _blog.clj_ in `src/clj/foo/example/db`.  
Add the following namespace declaration:

```
(ns foo.example.db.blog
  (:require [korma.core :refer [select]]
            [foo.example.db.entities :refer [post]]))
```

This imports the _select_ function and the _post_ entity that we just declared.

Now we will add a function to retrieve all posts:

```
(defn get-posts [ ]
  (select post))
```

This retrieves all rows and all fields from the post table. Korma supports a variety of functions to filter, ecxclude and limit your sql stataments. For more information, please look at the korma docs.

### Routes Namespace

The last step is to call our database function from our routes namespace that we created in the chapter before.  
Open `src/clj/foo/example/routes/blog.clj`. You can remove the _posts_ var for now.  
Add the following require statement to the _ns_ declaration:

```
[foo.example.db.blog :as b-db]
```

And change the _index-page_ function to look like this:

```
(defn index-page []
  (layout/render "blog/index.html" {:posts (b-db/get-posts)}))
```

As you can see we are calling our newly created function instead of referring to the var as we did before.

Switch to the browser and refresh the page. The content should look like this:

![Dynamic Posts Page](/img/model-01.png)

Also please note that no JVM restart is necessary here. During development closp will pick up changes and apply them accordingly, most of the times without manual interaction.

