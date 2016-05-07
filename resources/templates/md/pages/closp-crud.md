{:title "Closp Crud"
 :layout :page
 :page-index 2000}

In this chapter we will create a template for closp-crud which will be used for code generation.

## Creating a new template

Add a new file _post.edn_ in `env/dev/entities`. It should contain the following contents:

    {:name    "post"
     :columns [{:name "id" :type :int :null false :pk true :autoinc true}
               {:name "title" :type :varchar :max-length 100 :null false}
               {:name "content" :type :text}
               {:name "created_at" :type :time :null false}
               {:name "draft" :type :boolean :null false :default true}]}

This describes a new entity in closp where the structure is as follows:

- **:name** The name of the entity, is mapped to the table name in the database
- **:colums** A vector of column descriptions which are a map containing the following key / value pairs
- **:name** The name of the column, maps to the column name in the database table
- **:type** The type of the column, can be one of the following:
    - :int
    - :varchar
    - :char
    - :double
    - :decimal
    - :text
    - :time
    - :boolean
    - For a complete and current overview, please look here: <http://www.liquibase.org/documentation/column> at
    the types section.
- **:pk** If set to `true` it will generate a primary key description for the database
- **:autoinc** If set to `true` this will generate an auto incremeting column in the database
- **:null** Can be `true` or `false`, sets the column to nullable
- **:max-length** Will set the maximum length for a `varchar` type. Is mandatory if type is `varchar`
-- **:default** Can set a default value for the colum.

This is all thats needed to create the code for the _post_ entity.

## Creating the code

To generate the code run:

    lein run -m de.sveri.clospcrud.closp-crud/closp-crud -f env/dev/entities/post.edn

Afterwards you will find some new files in the appropriate places.

- **Migrations** The mirgraions file will reside in `resources/migrators/[db]/{date-}post.[up|down].sql`
- **src/clj/foo/example/db/post.clj** The code for accessing the database
- **src/clj/foo/example/routes/post.clj** The code where the different routes are defined
- **resources/templates/post** The html templates for _post_

## What is availabe

After the generation you will be able to create, edit and delete post, all the code to do this is in place.

## Activating the new routes

Before you can use the _post_ entity you will have to add the new route to the handler. To do so, open:
`src/clj/foo/example/components/handler.clj` and add the following require statement:

    [foo.example.routes.post :refer [post-routes]]

Additionally change this vec:

    [(cc-routes config) home-routes (user-routes config) base-routes]

to this:

    [(post-routes) (cc-routes config) home-routes (user-routes config) base-routes]

And finally switch to your repl and execute: `(reset)` to notify the webserver of the changed routes.
Afterwards open this page: <http://localhost:3000/post> and see the UI generated for the _post_ entity.

## Conclusion

If you followed the documentation you will see that most what we did in the chapters: `Model, Router, Template and update post`
can be done in a much faster way.
The code that was generated here can be used as a basic starting point and be adapted to your further needs.

## Restrictions

Closp-crud currently only works in a limited fashion. It does not support:

- Relationships (one-to-one, one-to-many)
- Backend validation
- View customization from template definition
- And a lot more

All of this may come some day or not, depending on my time. Pull requests are always welcome.