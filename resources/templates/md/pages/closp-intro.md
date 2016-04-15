{:title "Getting Started with Clojure Web Development"
 :layout :page
 :page-index 0}

## Prerequisites

To follow this tutorial you need to have installed:

- A JDK (Oracle, OpenJDK and IBM JDK > 1.6 are supported)
- Set the path to include the `bin` folder of the JDK
- Set the JAVA_HOME variable to point to your JDK
- Leiningen version > 2.5 - <http://leiningen.org/#install> (follow the instructions for a correct installation)
 

## Getting started

Open your command line and switch to a folder of your choice. Then execute the following commands:

> lein new closp _project-name_ -n _foo.example_  
> cd _project-name_   
> lein migrate  
> lein repl  
> in the repl enter: `(start-dev-system)`  
> lein figwheel (open a new command line for this, as `lein repl` will keep the existing one busy)

## Explanation

### lein new closp `project-name`-n `foo.example`

This creates a new clojure application in the folder `project-name`. It will download all the necessary files and create the appropriate folder structure.  

### lein migrate 

This will migrate the existing database files into a h2 database.  
The database files are stored in: `env/dev/entities`.  
And the new database will be created in: `db/`.

### lein repl

This will start a new REPL in your command line. After the REPL is up enter `(start-dev-system)` to start 
the web server on port _3000_.  
Your web application is now available at: <http://localhost:3000/>.


### lein figwheel

This command compiles the clojurescript files and opens a websocket to the browser. After switching
to <http://localhost:3000/example> you can see the page that serves the generated javascript on the
example page.


## Project Structure


```bash
|-- db
|-- env
|   |-- dev
|   |   |-- cljs - cljs files for development
|   |   |-- entities - entity descriptions for closp-crud
|-- integtest
|   |-- clj - selenium web tests
|-- resources
|   |-- sqlite - sqlite database migrations files
|   |-- public
|   |   |-- css
|   |   |-- img
|   |   |-- js
|   |-- templates - selmer html templates
|   |-- closp.edn - config file for closp
|   |-- closp-crud.edn - config file for closp-crud
|   |-- joplin.edn - config file for joplin (database migration)
|-- src
|   |-- clj - clojure source files
|   |-- cljc - clojure reader conditional source files
|   |-- cljs - clojurescript source files
|-- test - unit tests
```

Thats it, you can start coding now.